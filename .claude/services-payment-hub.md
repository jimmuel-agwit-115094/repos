# Payment Hub Services

All are .NET solutions with the standard NBS src layout. See [architecture.md](architecture.md) for layer descriptions.

## Banking (`Banking/`)

**Purpose:** Merchant account management, credit card transaction processing, transmission to payment processors.

**Key src modules:**
- `CaptureJob` / `TransmissionJob` / `NeedsVerificationJob` — background processing jobs
- `JobCoordinator` — orchestrates job execution
- `SsCore` — SS (settlement/submission) core logic
- `WebSpa` — Angular SPA for banking administration

**Dependencies:**
- EventHub (messaging)
- Encryption (PCI data)
- ChangeHistory (audit)

---

## Checkout (`Checkout/`)

**Purpose:** Order creation and checkout flow for NBS tenants.

**Key src modules:**
- `Checkout/` — core checkout domain
- `ExpireOrdersJob` — background job to expire stale orders
- `FeatureFlags` — LaunchDarkly flag definitions
- `SharedExtensions` — shared helpers within this solution

**Dependencies:**
- Banking, Encryption, EventHub, Payments, Remittance

---

## Payments (`Payments/`)

**Purpose:** Core payment processing; supports multiple UI surfaces (Angular SPA + Blazor).

**Key src modules:**
- `WebSpa` — Angular SPA
- `WebBlazor.Client` / `WebBlazor.Server` / `WebBlazor.Shared` — Blazor UI
- `NeedsVerificationJob` — processes payments pending verification
- `FeatureFlags` / `SharedConstants`

**Dependencies:**
- ChangeHistory, Encryption, EventHub, Banking, Remittance

---

## Remittance (`Remittance/`)

**Purpose:** Remittance processing and disbursement to NBS tenants.

**Key src modules:**
- `RemittanceJob` — main processing job
- `DbUpdater` — migration runner

**Dependencies:**
- EventHub, Encryption, ChangeHistory, Banking

---

## ScheduledPayments (`ScheduledPayments/`)

**Purpose:** Manages recurring/scheduled payment records and triggers.

**Key src modules:**
- `RecordJob` — job to record scheduled payment activity
- `Web.Common` — shared web utilities within solution

**Dependencies:**
- Encryption, EventHub (inferred from standard pattern)

---

## PaymentMethodSelector (`PaymentMethodSelector/`)

**Purpose:** Reusable web component + BFF for payment method selection UI; consumed by other service SPAs.

**Tech:** .NET BFF (`AspNetCore.Bff`) + Angular web component (`WebComponent`) + Angular SPA (`WebSpa`).

**CI:** Has separate npm (`azure-pipelines-ci-npm.yaml`) and .NET (`azure-pipelines-ci.yaml`) pipelines.
