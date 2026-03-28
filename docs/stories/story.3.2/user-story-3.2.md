# User Story 3.2: Product Category & Storage Classification

## 1. Meta Information
- **Epic:** [Epic 3: Unified Product Catalog (FL-6)](../../epics/EPICS.md)
- **Story Priority:** Medium (SHOULD — critical for warehouse zone enforcement and FDA compliance scope)
- **Story Point Estimation:** ~18h (3 tasks)
- **Status:** To Do
- **Jira:** FL-42

## 2. User Story
**As a** Procurement Manager,
**I want to** classify every product with a storage type (Cold Chain, Dry, Frozen, Ambient) and assign it to a named category,
**So that** the warehouse system can enforce correct storage zone assignment during receiving, trigger targeted FDA temperature log monitoring only for cold-chain products, and allow the team to filter the catalog by product family.

---

## 3. Business Context
Rwanda FDA compliance requires temperature logging for cold-chain produce. Without a `storage_type` classification on each product, Epic 9 (Compliance) cannot know which products require temperature monitoring, and Epic 4 (Receiving) cannot validate that a cold-chain product was received in the correct warehouse zone. The `storage_type` field is a cross-cutting architectural concern that enables both food safety and regulatory compliance downstream.

---

## 4. Acceptance Criteria (BDD Format)

### Scenario 1: Cold chain product enforces zone compliance downstream
**Given** the Procurement Manager creates a product "Avocado" with `storage_type: COLD_CHAIN`
**When** a batch of Avocado is received in Epic 4
**Then** the receiving system checks the delivery zone against the product's `storage_type` and blocks confirmation if the zone is a dry storage zone
**And** Epic 9 includes this product in the temperature monitoring alert scope.

### Scenario 2: Category management
**Given** an Admin navigates to the Category Management page
**When** they create a new category "Root Vegetables"
**Then** it immediately appears as a selectable option in the Product Create/Edit form's category dropdown
**And** cannot be deleted while any active product references it (returns `409 Conflict`).

### Scenario 3: Filtering by storage type in the catalog
**Given** the Product Catalog list view is open
**When** the user selects "COLD_CHAIN" from the Storage Type filter
**Then** only cold-chain products are returned in the paginated results with the COLD_CHAIN blue badge visible on each row.

---

## 5. Technical Tasks (Developer Implementation)
- [ ] **Task 1:** [Add storage_type ENUM to Product_Master schema and update form UI.](./task-3.2.1.md) → Jira: FL-48
- [ ] **Task 2:** [Build Category Management API and admin UI.](./task-3.2.2.md) → Jira: FL-49
- [ ] **Task 3:** [Document storage_type dependency contract for Epic 4 and Epic 9.](./task-3.2.3.md) → Jira: FL-50

---

## 6. Edge Cases & Risk Mitigation
- **`storage_type` changed on an active product with existing batches**
  - *Mitigation:* System displays a warning modal: "This product has X active batches in [old zone]. Warehouse Manager must physically relocate them." The system does not auto-move existing batch records — this requires a manual warehouse operation.
- **Category deleted while products still reference it**
  - *Mitigation:* Soft-delete only (`is_active: false`). Backend returns `409 Conflict` if any active product references the category. PM must reassign affected products first.

---

## 7. Dependencies
- Blocked by Story 3.1 (Task FL-31 must already have `storage_type` in the `Product_Master` schema).
- Provides data contract for Epic 4 (Receiving) and Epic 9 (Compliance/Guardrail).
- Task 3 (FL-50) must be reviewed by Engineering lead before Epic 4 sprint begins.

---

## 8. PM Open Questions / Clarifications
- **userAskQuestion:** Should `storage_type` be a required field on product creation from day one, or optional with a default of `DRY` to avoid blocking the catalog population during initial data entry?
- **userAskQuestion:** For existing products in the system before this story deploys — what is the agreed-upon default `storage_type` for the data migration? (Suggested: `DRY` as the safest default, reclassified manually.)
