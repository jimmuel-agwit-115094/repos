# Architecture

## Overview

NBS (Nelnet Business Services) platform — multi-tenant payment processing system. All repos live under `C:\neldevsrc\repos`. Each repo is an independent .NET solution or Angular library deployed to Kubernetes.

## Service Groups

### Payment Hub Services
Core payment processing pipeline. Tight inter-service dependencies.

| Repo | Role |
|------|------|
| `Banking` | Merchant accounts, credit card processing, transmission jobs |
| `Checkout` | Order creation and checkout flow |
| `Payments` | Payment processing, needs-verification workflow |
| `Remittance` | Remittance processes, disbursements |
| `ScheduledPayments` | Recurring/scheduled payment records and jobs |
| `PaymentMethodSelector` | Web component + BFF for selecting payment methods |

### Platform/Infrastructure Services
Shared services consumed by most Payment Hub services.

| Repo | Role |
|------|------|
| `Encryption` | Encryption API — PCI-scoped data encryption |
| `EventHub` | Azure Event Hub wrapper — cross-service messaging bus |
| `ChangeHistory` | Audit trail API + Angular SPA |
| `Passport` | OAuth identity + authorization (API keys, scopes, roles) |
| `Tenant` | Tenant registry, caching, context propagation |
| `Menu` | Navigation menu API + SPA |
| `CustomerServicePortal` | Internal support portal (has own `.claude/` docs) |

### Angular Frontend Libraries
Shared UI libraries published as npm packages; consumed by service SPAs.

| Repo | Role |
|------|------|
| `xlr8AngularToolkit` | Component library (`ng-xlr8-toolkit` + styles) |
| `xlr8PageTemplate` | Page layout template (`xlr8-page-template`) |
| `Framework.Web.AngularTesting` | Shared Angular test utilities (Karma + Playwright) |

## Common .NET Solution Layout

Each backend service follows this src layout:

```
src/
  Contracts/           # Public DTOs and event contracts (no external deps)
  Client.Contracts/    # Typed client interfaces
  Client/              # NSwag-generated HTTP client
  Accessors/           # Data access (MongoDB, SQL, external HTTP calls)
  Services/            # Business logic layer
  BackgroundServices/  # Hosted services (event consumers)
  WebApi/              # ASP.NET Core API
  WebSpa/              # Angular SPA (BFF-hosted)
  DbScripts/           # MongoDB seed/migration scripts
  *Job/                # Azure Functions or console job processes
```

> Some repos also have `WebBlazor.*` (Payments) or `WebComponent/` (PaymentMethodSelector).

## Hosting

- All services containerized and deployed to **Kubernetes** via Helm charts (`charts/` folder per repo).
- CI/CD via **Azure DevOps Pipelines** (`azure-pipelines-*.yml/yaml` per repo).
- Infrastructure: **MongoDB Atlas**, **Redis**, **Azure Event Hubs**, **AWS**.

## Auth

All services authenticate via **Passport** — NBS's internal OAuth2/OpenID provider.
- API access requires API keys with scopes (e.g., `Passport.Secured.API`).
- Angular SPAs use BFF pattern — auth handled server-side.

## Feature Flags

`Nbs.Framework.FeatureFlags` (LaunchDarkly-backed) used across most services. Flags are tenant-aware.

## Observability

- **DataDog** for APM and RUM.
- **Service Now** CMDB integration via CI pipelines.
