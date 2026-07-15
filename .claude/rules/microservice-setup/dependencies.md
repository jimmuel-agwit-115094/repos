# Inter-Service Dependencies

## Dependency Graph

```
Checkout ──────────────► Banking ──────► Encryption
    │                      │               ▲
    │                      ▼               │
    ├──────────────► Payments ─────────────┤
    │                  │   │               │
    │                  │   └──────────────►│
    │                  ▼                   │
    ├──────────────► Remittance ───────────┤
    │                                      │
    ├──────────────► Encryption ───────────┘
    │
    └──────────────► EventHub ◄─── (all services publish/consume)
                       ▲
          ChangeHistory─┘  (all services write audit events)

ScheduledPayments ──────► Encryption
                    ──────► EventHub

PaymentMethodSelector ──► (BFF calls: Banking / Payments — inferred)

All services ───────────► Passport (auth)
All services ───────────► Tenant (tenant context)
All SPAs ───────────────► Menu (navigation)
```

## Required Local Running Order

When running a service locally with E2E tests, start dependencies first:

| Service | Start First |
|---------|-------------|
| Banking | EventHub, Encryption, ChangeHistory |
| Checkout | Banking, Encryption, EventHub, Payments, Remittance |
| Payments | ChangeHistory, Encryption, EventHub, Banking, Remittance |
| Remittance | EventHub, Encryption, ChangeHistory, Banking |
| ScheduledPayments | EventHub, Encryption |

## NuGet / npm Package Dependencies

- All .NET services consume `Passport.Client` and `Passport.Client.AspNetCore` from Passport.
- All .NET services consume `Tenant.Client` and `Tenant.TenantContext.AspNetCore` from `xlr8-app-tenant`.
- All Angular SPAs consume `ng-xlr8-toolkit`, `ng-xlr8-toolkit-styles` from xlr8AngularToolkit.
- All Angular SPAs consume `xlr8-page-template` from xlr8PageTemplate.
- SPA test projects consume `Framework.Web.AngularTesting` packages.
- Service SPAs consume `Nbs.Branding.Client` from `xlr8-app-branding` for tenant theming.
