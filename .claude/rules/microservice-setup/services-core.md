# Core / Platform Services

Shared infrastructure services consumed by Payment Hub and other services. Treat these as upstream dependencies.

---

## Encryption (`Encryption/`)

**Purpose:** PCI-scoped data encryption/decryption API. All services that handle sensitive payment data (card numbers, bank accounts) call this service.

**Stack:** .NET API only — no SPA (has `WebSpa` folder but minimal).

**Note:** Locally required by: Banking, Checkout, Payments, Remittance, ScheduledPayments.

---

## EventHub (`EventHub/`)

**Purpose:** Provides publish/subscribe messaging across all NBS services. Using API query and Webhook

**Key src modules:**
- `BackgroundServices` — event consumers
- `Client` / `Client.Contracts` — typed client for other services to consume
- `WebSpa` — admin/monitoring SPA

**Note:** Nearly every service depends on EventHub for async domain events.

---

## ChangeHistory (`ChangeHistory/`)

**Purpose:** Centralized audit trail. Services record state changes here.

**Stack:** .NET WebApi + Angular SPA (workspace in `src/workspace`).

**Note:** Has both npm and .NET CI pipelines.

---

## Passport (`Passport/`)

**Purpose:** NBS identity provider — OAuth2/OpenID Connect. Manages API keys, scopes, role assignments for all services.

**Key src modules:**
- `Passport.Web` — main auth server
- `Passport.AdminWeb` — admin SPA for managing keys/roles
- `Passport.Client` / `Passport.Client.AspNetCore` — NuGet client packages
- `Passport.AuthorizationCache.*` — Redis/memory/Passport-backed auth cache implementations
- `Passport.BackgroundServices` — background token/cache refresh
- `PassportDb` / `PassportDbInstaller` — database setup
- `TestIDP` — test identity provider for local dev

**Note:** Upstream dependency of all services. API keys must be provisioned in Passport before a service can call other services.

---

## Tenant (`Tenant/`)

**Purpose:** Tenant registry and context. Tracks all NBS tenants; provides `TenantContext` middleware for per-request tenant resolution.

**Key src modules:**
- `Caching` / `Caching.AspNetCore` / `Client.Caching` — layered tenant caching
- `TenantContext.AspNetCore` — ASP.NET Core middleware
- `FeatureFlags`

**Note:** Tenant context is injected into every service request. Changes here can have wide impact.

---

## Menu (`Menu/`)

**Purpose:** Navigation menu API + SPA. Provides the top-nav menu shown in all NBS service SPAs.

**Stack:** .NET WebApi + Angular SPA.

**Note:** No `Accessors` or `BackgroundServices` — relatively thin service.

---

## CustomerServicePortal (`CustomerServicePortal/`)

**Purpose:** Internal support portal for NBS tenant administration.

**Stack:** .NET 10 + Angular 20, MongoDB Atlas, Redis.

**Note:** Has its own complete `.claude/` documentation set. See `CustomerServicePortal/CLAUDE.md` for full details. Was scaffolded from the standard NBS service template.
