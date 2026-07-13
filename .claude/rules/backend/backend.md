# Backend Architecture Map

## Overview

Root: `C:\neldevsrc\repos`. Each subdirectory is an independent .NET solution deployed as a Kubernetes pod.

**Stack:** .NET 9/10, C#, MongoDB Atlas (primary), SQL Server (Passport only), Redis (auth/tenant cache), Azure Event Hubs (cross-service), AWS SQS (Banking batch jobs), Snowflake (Payments reporting only).

**Every service uses the same layer structure:**
```
WebApi → Services → Accessors → (MongoDB | external HTTP clients)
```
No skipping layers. Accessors never reference Services.

---

## Project Responsibilities

| Service | Owns | Key NuGet clients consumed by others |
|---|---|---|
| **Banking** | Payment gateway (PaymentSpring CC + BT), payment method tokenization, bank calendars, return files, risk scoring | `Nbs.Banking.Client` |
| **Checkout** | Payment session lifecycle, order creation, submission to Payments, refunds, third-party payment flows | `Nbs.Checkout.Client` |
| **Payments** | Core payment engine — validates, routes to Banking, enforces payment controls, manages profiles, receipts, remittance coordination | `Nbs.Payments.Client` |
| **Remittance** | Collects completed payment txns, batches disbursements to tenants via Banking BT transfers | `Nbs.Remittance.Client` |
| **ScheduledPayments** | Recurring/installment schedule state machine, submits due payments to Payments on schedule | `Nbs.ScheduledPayments.Client` |
| **EventHub** | Internal webhook bus — persists events, manages subscriptions, delivers to subscriber URLs with retry | `Nbs.EventHub.Client` |
| **xlr8-app-tenant** | Tenant registry, `TenantContext` middleware (injected into all other services), publishes `tenant-changed` | `Nbs.Tenants.Context.AspNetCore` |
| **Passport** | OAuth2/OIDC provider (Duende IdentityServer), users, roles, API keys, `[TenantSecuredFunction]` attribute | `Nbs.Passport.Client.AspNetCore` |
| **ChangeHistory** | Centralized audit trail — all services post changesets here | `Nbs.ChangeHistory.Client` |
| **Encryption** | PCI-scoped encrypt/decrypt — services store only an `encryptedItemId` reference | `Nbs.Encryption.Client` |
| **Menu** | Role+tenant aware navigation menus for all SPAs; no Accessors, pure computation | `Nbs.Menu.Client` |
| **Contact** | Tenant contact CRUD with diff-based change history | `Nbs.Contact.Client` |
| **CustomerServicePortal** | Internal support portal — own read-optimized MongoDB projections updated via EventHub; not a public API surface |  |

---

## Request / Processing Flows

### Synchronous HTTP request (any service)
```
HTTP → WebApi Controller
         → [TenantSecuredFunction] auth filter (Passport cache lookup)
         → TenantContext middleware (resolves ITenantContext from route)
         → Service method
             → Accessor (MongoDB read/write)
             → Optional: HTTP client to downstream service
             → Optional: IPublisherFactory.GetPublisher(topic).Publish(msg)  [in-memory saga]
         ← Response DTO
```

### Cross-service payment lifecycle
```
Checkout POST /v3/{tid}/payment-session/{id}/submit
  └─ calls Payments POST /v2/{tid}/payment
       └─ calls Banking POST /v1/{tid}/credit-card   [CC]
            └─ Banking publishes SQS: credit-card-tran-ready-for-capture
                 └─ Banking CaptureJob Worker.ExecuteAsync()
                      └─ publishes EventHub: banking-credit-card-transaction-completed
                           └─ Payments BackgroundServices: CreditCardTransactionCompletionHandler
                                └─ publishes EventHub: payments-payment-completed
                                     ├─ Checkout: PaymentCompletionHandler → marks session complete
                                     ├─ ScheduledPayments: PaymentCompletedHandler → marks installment paid
                                     └─ Remittance: BankTransferTransactionCompletionHandler → creates txn record
                                          └─ RemittanceJob → Banking BT transfer → SQS: remittance-payment-tx-completed
                                               └─ Payments: RemittancePaymentTransactionCompletionHandler
```

### Background job execution (all services)
```
Job Worker implements IHostLifetimeBackgroundService
  → ExecuteAsync() called once
  → calls XxxJobService.RunForGlobal(...)
  → host stops automatically
```
Exception: `ScheduledPayments/RecordJob` uses base `BackgroundService` and calls `_hostLifetime.StopApplication()` manually.

### EventHub message handling
```
BackgroundServices host → IMessageHandlerOf<TMessage> implementations
  → [MessageHandlerOf("topic-name")] attribute routes inbound message
  → handler calls Service methods
  → Service may publish in-memory topics to continue internal saga
```

---

## Responsibility Boundaries

| Concern | Where it lives | Where it does NOT live |
|---|---|---|
| Payment gateway calls (PaymentSpring) | `Banking/src/Accessors/` | Payments, Checkout |
| Payment orchestration | `Payments/src/Services/PaymentService.cs` | Banking, Checkout |
| Order/session state | `Checkout/src/Accessors/PaymentSessions/PaymentSessionV3Accessor.cs` | Payments, Banking |
| Schedule state machine | `ScheduledPayments/src/Services/ScheduleService.cs` | Payments, Checkout |
| Tenant context resolution | `xlr8-app-tenant/src/TenantContext.AspNetCore/` (NuGet middleware) | Every other service (consumers only) |
| Auth enforcement | `Passport/src/Passport.Client.AspNetCore/` (`[TenantSecuredFunction]`) | Business logic services |
| Audit trail | `ChangeHistory/src/Services/ChangeSetService.cs` | Each service (async publish only) |
| Sensitive data (PAN, account numbers) | `Encryption/src/Accessors/EncryptedItemAccessor.cs` | All other services (store `encryptedItemId` only) |
| Webhook delivery | `EventHub/src/Services/Handlers/EventReadyToTransmitHandler.cs` | Application services |
| CSP read projections | `CustomerServicePortal/src/Accessors/` | Source-of-truth services (Passport, PeopleManagement) |

---

## Key Patterns

### DI Registration via Scrutor + Attributes
```csharp
// Mark a class for DI scan — no manual registration needed
[RegisterService]          // Transient; registered in all hosts
[RegisterBackgroundService] // Registered only in BackgroundServices host
```
`services.Scan(...)` in each host's `Startup.cs` picks them up. The same `Services.csproj` is referenced by WebApi, BackgroundServices, and Job hosts; attributes control which host gets which class.

### Three-Tier Messaging
| Tier | Transport | Scope | Typical topic file |
|---|---|---|---|
| Cross-service | Azure EventHub | Platform-wide pub/sub | `Client.Contracts/XxxEventTypes.cs` |
| Batch job triggers | AWS SQS | Banking jobs only | `Banking/src/Services/Messaging/SqsTopics.cs` |
| Saga steps | In-Memory | Within one service process | `Contracts/Messaging/InMemoryTopics.cs` |

All tiers use the same `IPublisherFactory.GetPublisher(topicId).Publish()` API.

### Transactional Outbox (MongoDB)
```csharp
// BackgroundServices Startup.cs — typical pattern
services.AddMessaging()
    .WithMongoDbOutbox(_configuration)
    .WithEventHub(_configuration)
    .WithAws(_configuration);
```
Domain write + outbox insertion are atomic. Background process delivers and retries. Always use outbox for cross-service EventHub/SQS publishes — never publish directly without it.

### Auth on Every Controller
```csharp
[Authorize]
[TenantSecuredFunction(SecuredFunctions.ManagePayments)]
public class PaymentController : ControllerBase { ... }
```
`[TenantSecuredFunction]` defined in `Passport/src/Passport.Client.AspNetCore/Authorization/Tenant/TenantSecuredFunctionAttribute.cs`. Secured function IDs are constants in each service's `SharedConstants/SecuredFunctions.cs`.

### Typed HTTP Client Registration
```csharp
services.AddServiceConnectionHttpClient<IPaymentClient, PaymentClient>("PaymentsClient")
    .WithTransientErrorRetry()
    .EnableCompression()
    .ForwardTimeTraveling();
```
Client URL configured in `appsettings.json` service connections section. All inter-service HTTP uses NSwag-generated clients from `{Service}/src/Client/`.

### MongoDB Accessor Base Classes
| Base class | Use when |
|---|---|
| `BaseMongoDbAccessor<TDoc>` | Standard CRUD |
| `BaseStrongKeyedMongoDbAccessor<TDoc, TKey>` | Composite key lookups (e.g., `PaymentSessionKey`) |
| `BaseRevisionAccessorOfT` | Optimistic concurrency (ScheduledPayments schedules) |

Each accessor has a paired `*Mapper.cs` converting `*Doc` ↔ domain object.

### Feature Flags
```csharp
// Each service defines its own interface
public interface IPaymentsFeatureFlags {
    bool IsNewPaymentFlowEnabled(ITenantContext context);
}
// Evaluated with tenant as LaunchDarkly targeting key
```
Flag interface lives in `{Service}/src/FeatureFlags/`.

### MongoDB Migrations
`{Service}/DbScripts/` — timestamped Node.js scripts in `InstallScripts/`, `SeedScripts/`, `DatafixData/`. Run manually or via CI. Remittance also has a `DbUpdater` .NET console app (`dotnet run`).

---

## Dependency Direction

```
                   ┌─────────────────────────────────┐
                   │   xlr8-app-tenant  ·  Passport  ·  Menu  │  (Platform layer — all services depend on these)
                   └─────────────────────────────────┘
                              ↑         ↑
        ┌──────────────────────────────────────────┐
        │  ChangeHistory  ·  Encryption  ·  EventHub │  (Shared infrastructure)
        └──────────────────────────────────────────┘
                              ↑
        ┌─────────────────────────────────────────────────┐
        │  Banking  →  Payments  →  Checkout              │  (Payment domain — left-to-right dependency)
        │              Payments  →  Remittance             │
        │              Payments  →  ScheduledPayments      │
        └─────────────────────────────────────────────────┘
                              ↑
        ┌─────────────────┐
        │ CustomerService │  (Consumer — reads projections, calls upstream APIs)
        │ Portal          │
        └─────────────────┘
```

**Rules:**
- Banking has no upstream service dependencies (except platform layer).
- Checkout calls Payments; Payments calls Banking — never the reverse.
- ScheduledPayments calls Payments to submit; Payments does not call ScheduledPayments.
- CSP calls many services but no service calls CSP.

---

## Where to Look First

| Task | Start here |
|---|---|
| Add a new API endpoint | Find existing controller in `{Service}/src/WebApi/Controllers/`, copy the pattern |
| Add business logic | `{Service}/src/Services/` — find the relevant `*Service.cs` |
| Add a MongoDB read/write | `{Service}/src/Accessors/` — find the relevant `*Accessor.cs` and `*Doc.cs` |
| Handle an inbound EventHub event | `{Service}/src/Services/Messaging/` — find existing `*Handler.cs` with `[MessageHandlerOf]` |
| Publish a new EventHub event | `{Service}/src/Client.Contracts/XxxEventTypes.cs` for topic name; add to outbox publish chain |
| Add a new in-memory saga step | `{Service}/src/Contracts/Messaging/InMemoryTopics.cs` for topic; new handler with `[MessageHandlerOf]` |
| Understand payment flow | `Payments/src/Services/PaymentService.cs` — central orchestrator |
| Understand checkout flow | `Checkout/src/Services/PaymentSessionSubmitService.cs` + `PaymentSessionCompletionService.cs` |
| Understand job execution | Any `{Service}/src/{JobName}/Worker.cs` — all follow same `IHostLifetimeBackgroundService` pattern |
| Understand tenant resolution | `xlr8-app-tenant/src/TenantContext.AspNetCore/` middleware |
| Understand auth checks | `Passport/src/Passport.Client.AspNetCore/Authorization/Tenant/TenantSecuredFunctionAttribute.cs` |
| Find secured function IDs | `{Service}/src/SharedConstants/SecuredFunctions.cs` |
| Find event topic names | `{Service}/src/Client.Contracts/XxxEventTypes.cs` (EventHub) or `InMemoryTopics.cs` (in-process) |
| Find package versions | `{Service}/Directory.Packages.props` — all versions centrally managed here |

---

## Critical Rules

- **Never skip layers.** WebApi → Services → Accessors. Accessors must not reference Services.
- **Accessors own all MongoDB access.** Services never call MongoDB directly.
- **Always use Outbox for cross-service publishing.** `.WithMongoDbOutbox(...)` in BackgroundServices startup. Direct EventHub publish without outbox loses messages on crash.
- **Every controller action needs `[TenantSecuredFunction]`.** No unauthenticated endpoints except health checks and webhooks.
- **Store `encryptedItemId`, never plaintext.** Any PAN, bank account number, or sensitive credential must go through the Encryption service.
- **CSP's API is internal.** No other service should call CustomerServicePortal endpoints.
- **Banking owns the gateway.** Payments and Checkout must not call PaymentSpring directly — they call Banking.
- **`[MessageHandlerOf]` crashes startup if the topic is not registered.** Do not add a handler for a topic that isn't wired in the host's messaging startup. Use `[RegisterService]` (not `[RegisterBackgroundService]`) for CSP outbox handlers.
- **Passport uses SQL Server, not MongoDB.** All Passport accessor work uses Dapper row DTOs, not `BaseMongoDbAccessor`.
- **No `Version=` in `.csproj` files.** All NuGet versions are in `Directory.Packages.props` — adding a version inline will conflict.

---

## Related Rules

- [Architecture overview](../architecture/architecture.md) — service groups, hosting, auth, feature flags
- [Conventions](../conventions/conventions.md) — C#, Angular/TS naming, CI/CD, DB patterns
- [Service dependencies](../microservice-setup/dependencies.md) — local startup order, shared packages
- [Unit tests](../tests/backend-service-test.md) — xUnit, Moq, FluentAssertions patterns
- [Integration tests](../tests/integration-test.md) — WebApplicationFactory, in-memory messaging
- [Payment Hub services](../microservice-setup/services-payment-hub.md) — Banking, Checkout, Payments, Remittance, ScheduledPayments detail
- [Core services](../microservice-setup/services-core.md) — Encryption, EventHub, ChangeHistory, Passport, Tenant, Menu, CSP detail
