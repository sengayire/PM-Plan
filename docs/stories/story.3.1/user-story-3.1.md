# User Story 3.1: Master Product (SKU) Management

## 1. Meta Information
- **Epic:** [Epic 3: Unified Product Catalog (FL-6)](../../epics/EPICS.md)
- **Story Priority:** High (MUST — Foundation for Inventory, QC, and warehouse zone enforcement)
- **Story Point Estimation:** ~28h (4 tasks)
- **Status:** To Do
- **Jira:** FL-29

## 2. User Story
**As a** Procurement Manager,
**I want to** define and maintain products (SKUs) along with their specific shelf-life characteristics, storage temperature requirements, and storage type classification,
**So that** the system can automate expiry date calculations during receiving flows, enforce FIFO rules, assign products to the correct warehouse zone, and satisfy Rwanda FDA compliance requirements.

---

## 3. Business Context
AgriFlow operates with highly perishable goods spanning multiple storage environments — cold chain (tomatoes, avocados), dry (maize, beans), and frozen. Without a standardized product master that includes storage type, the warehouse cannot enforce zone compliance, cannot scope FDA temperature log monitoring correctly, and cannot automate batch expiry calculations. This is the foundational data layer that every downstream epic depends on.

---

## 4. Acceptance Criteria (BDD Format)

### Scenario 1: Creating a new product with full classification
**Given** the Procurement Manager is logged into the Master Data dashboard
**When** they submit the "Create Product" form with "Tomatoes", shelf-life "4 days", temperature "4°C - 8°C", storage type "COLD_CHAIN", UoM "KG"
**Then** the system creates the product entity
**And** it immediately appears in supplier mapping flows, receiving dropdowns, and inventory views.

### Scenario 2: Validation prevents invalid data
**Given** a user is configuring a new product
**When** they enter a negative `shelf_life_days`, leave `unit_of_measure` blank, or set `storage_temp_min` > `storage_temp_max`
**Then** the UI prevents submission with field-level error messages
**And** the backend returns `400 Bad Request` if the UI validation is bypassed.

### Scenario 3: Catalog browsing with storage type filtering
**Given** multiple products exist across storage types
**When** the Procurement Manager navigates to "Product Catalog"
**Then** they see a paginated, searchable list with color-coded storage type badges (COLD_CHAIN=blue, DRY=green, FROZEN=cyan, AMBIENT=gray)
**And** they can filter the list by storage type, category, or active status.

---

## 5. Technical Tasks (Developer Implementation)
- [ ] **Task 1:** [Create Product_Master PostgreSQL schema (includes storage_type ENUM).](./task-3.1.1.md) → Jira: FL-31
- [ ] **Task 2:** [Build Product Catalog REST API (NestJS CRUD).](./task-3.1.2.md) → Jira: FL-32
- [ ] **Task 3:** [Build Product Catalog data table UI (Next.js).](./task-3.1.3.md) → Jira: FL-33
- [ ] **Task 4:** [Build Create/Edit Product form UI (Next.js).](./task-3.1.4.md) → Jira: FL-34

---

## 6. Edge Cases & Risk Mitigation
- **Shelf-life rules change after active transactions**
  - *Mitigation:* Modifications to `shelf_life_days` apply ONLY to future receiving batches. Existing inventory retains expiry dates calculated at time of receipt.
- **Product line discontinued**
  - *Mitigation:* Hard deletes are forbidden. `is_active: false` soft-delete hides the product from future dropdowns while preserving all historical ledger entries and audit trail.

---

## 7. Dependencies
- Blocked by Epic 1 (IAM) — RBAC middleware must exist to guard mutation endpoints.
- Blocks Story 2.2 (Supplier-Product Price Mapping) — `Product_Master` must exist for FK references.
- Blocks Epic 4 (Smart Receiving) — receiving flow reads `shelf_life_days` and `storage_type` to calculate expiry and validate warehouse zone.
- Blocks Epic 9 (Compliance) — temperature log alerts scoped to products where `storage_type IN ('COLD_CHAIN', 'FROZEN')`.

---

## 8. PM Open Questions / Clarifications
- **userAskQuestion:** Should the `unit_of_measure` ENUM (KG/PCS/BUNDLE) be extended to include LITRE, CRATE for future product lines, or is a configurable UoM management table needed before Phase 2?
- **userAskQuestion:** For existing products before `storage_type` migration: default to `DRY` unless reclassified — confirm with Warehouse Manager before running migration.
