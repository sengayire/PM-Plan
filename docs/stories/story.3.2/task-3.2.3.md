# Task 3.2.3: Document storage_type Dependency Contract for Epic 4 and Epic 9

## 1. Meta Information
- **Parent Story:** [User Story 3.2: Product Category & Storage Classification](./user-story-3.2.md)
- **Epic:** [Epic 3: Unified Product Catalog (FL-6)](../../epics/EPICS.md)
- **Jira:** FL-50
- **Estimate:** 3h
- **Assignee:** Engineering Lead (must be reviewed by Epic 4 and Epic 9 leads)
- **Status:** To Do

## 2. Objective
Write a formal dependency contract document that specifies how Epic 4 (Smart Receiving) and Epic 9 (Loss & Compliance Guardrail) must consume the `storage_type` field from `Product_Master`. This document prevents integration-time surprises and must be reviewed and signed off by the relevant epic leads before their sprint begins.

## 3. Deliverable: `docs/contracts/storage-type-contract.md`

The document must cover:

### 3.1. Contract Overview
- **Provider:** Epic 3 (Unified Product Catalog) — `Product_Master.storage_type` ENUM
- **Consumers:** Epic 4 (Smart Receiving & Traceability), Epic 9 (Loss & Compliance Guardrail)
- **Version:** 1.0
- **Review date:** Before Epic 4 sprint kickoff

### 3.2. ENUM Values & Definitions
Document all four values with their operational meaning and warehouse zone implications:

| Value | Warehouse Zone | Temperature | FDA Monitoring |
|-------|---------------|-------------|----------------|
| `COLD_CHAIN` | Cold Storage Zone (2°C – 8°C) | Refrigerated | Required |
| `FROZEN` | Freezer Zone (< -18°C) | Frozen | Required |
| `DRY` | Dry Goods Zone (ambient) | Ambient | Not required |
| `AMBIENT` | General Storage | Room temp | Not required |

### 3.3. Epic 4 (Receiving) Integration Rules
Specify the exact business rules Epic 4 must enforce using `storage_type`:

1. **Zone Validation on Receiving:** When a delivery batch is confirmed in Epic 4, the system must read `Product_Master.storage_type` for the received product and validate the assigned warehouse zone:
   - `COLD_CHAIN` or `FROZEN` → batch must be assigned to a zone of matching type; reject with `400` if zone is `DRY` or `AMBIENT`
   - `DRY` or `AMBIENT` → no zone restriction enforced

2. **API Contract:** Epic 4's receiving endpoint must call `GET /products/:id` to retrieve `storage_type` before zone validation. Epic 3 guarantees `storage_type` will never be null for any `is_active: true` product.

3. **Change propagation:** If `storage_type` is changed on a product after batches have been received, Epic 4 does **not** auto-relocate existing batches. A manual warehouse operation is required. Epic 3's change warning (FL-48) surfaces this to the Procurement Manager.

### 3.4. Epic 9 (Compliance) Integration Rules
Specify the exact business rules Epic 9 must enforce using `storage_type`:

1. **Temperature Log Scope:** Epic 9's temperature monitoring alerts must be scoped to products where:
   ```sql
   storage_type IN ('COLD_CHAIN', 'FROZEN')
   ```
   Products with `DRY` or `AMBIENT` storage type must be excluded from temperature log monitoring to prevent alert fatigue.

2. **FDA Compliance Report:** The Rwanda FDA audit export (Epic 9) must include a `storage_type` column from `Product_Master` for all batches in the reporting period.

3. **Alert Triggering:** When a temperature log exceeds the acceptable range for a product, Epic 9 must cross-reference `Product_Master.storage_temp_min` and `Product_Master.storage_temp_max` (provided by Epic 3) for the specific product, not a generic threshold.

### 3.5. Default Value Contract
- **For new products:** `storage_type` is a required field on `POST /products` — no default assumed by consumers.
- **For migrated products (pre-story deployment):** All rows backfilled to `DRY` via `BackfillStorageTypeToDry` migration (FL-48). Warehouse Manager must manually reclassify all non-dry products before Epic 4 sprint begin. This is a **hard prerequisite** for Epic 4 sprint readiness.

### 3.6. Review & Sign-Off
The document must include a sign-off section:
- Epic 4 Lead: [Name] — reviewed and approved
- Epic 9 Lead: [Name] — reviewed and approved
- Engineering Lead: [Name] — approved before Epic 4 sprint kickoff

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `docs/contracts/storage-type-contract.md` file created with all sections above
- [ ] ENUM table documents all 4 values with zone, temperature, and FDA monitoring columns
- [ ] Epic 4 zone validation rules are unambiguous (specific error codes specified)
- [ ] Epic 9 SQL filter is documented explicitly (`storage_type IN ('COLD_CHAIN', 'FROZEN')`)
- [ ] Default value contract and migration backfill approach is documented
- [ ] Sign-off section populated before Epic 4 sprint kickoff
- [ ] Document reviewed by Epic 4 and Epic 9 leads (tracked via Jira FL-50 review workflow)

## 5. Dependencies
- **Requires:** FL-48 (storage_type ENUM confirmed in schema — verify field is live before writing contract)
- **Requires:** FL-49 (Category Management — mention categories are available but not a dependency for Epic 4/9)
- **Must be reviewed before:** Epic 4 sprint kickoff (FL-7 sprint planning)
- **Must be reviewed before:** Epic 9 sprint kickoff (FL-12 sprint planning)
