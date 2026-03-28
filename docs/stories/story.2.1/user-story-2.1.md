# User Story 2.1: Supplier Onboarding & Profile Management

## 1. Meta Information
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Story Priority:** High (MUST — prerequisite for price mapping and receiving workflows)
- **Story Point Estimation:** ~26h (5 tasks)
- **Status:** To Do
- **Jira:** FL-40

## 2. User Story
**As a** Procurement Manager,
**I want to** register new suppliers in the system with their full profile and attach their signed commercial agreement document,
**So that** the warehouse only ever accepts deliveries from verified, contracted suppliers and Finance has a complete, auditable record of every supplier relationship.

---

## 3. Business Context
AgriFlow's "front gate" rule: no unverified supplier should ever deliver produce to the warehouse. Without a digital supplier registry with attached contracts, receiving clerks have no way to verify whether a delivery is from an approved partner, and Finance has no documented basis for the agreed pricing. This story establishes that registry as the gating prerequisite for the full supply chain.

---

## 4. Acceptance Criteria (BDD Format)

### Scenario 1: Registering a new supplier
**Given** the Procurement Manager completes the "Register Supplier" form with name, contact, location, and bank details
**When** they submit
**Then** the system creates the supplier with `is_active: true`
**And** the new supplier immediately appears in the Supplier Directory and is selectable in product price mapping and receiving workflows.

### Scenario 2: Attaching a contract document
**Given** a registered supplier's profile page is open
**When** the Procurement Manager uploads a PDF contract document
**Then** the file is stored in AWS S3 under `supplier-contracts/{supplierId}/` with private ACL
**And** a permanent, viewable link is attached to the supplier's profile accessible via a 1-hour pre-signed URL.

### Scenario 3: Suspending a supplier
**Given** a supplier needs to be temporarily suspended
**When** the Procurement Manager toggles `is_active: false` with a mandatory suspension reason note
**Then** the supplier is hidden from all future receiving workflows and price mapping dropdowns
**And** their full profile, documents, and transaction history remain intact and searchable by Admin and Auditor roles.

---

## 5. Technical Tasks (Developer Implementation)
- [ ] **Task 1:** [Create Suppliers table PostgreSQL schema and migration.](./task-2.1.1.md) → Jira: FL-35
- [ ] **Task 2:** [Build Supplier Registration & Profile REST API (NestJS).](./task-2.1.2.md) → Jira: FL-43
- [ ] **Task 3:** [Build Supplier Directory list UI (Next.js).](./task-2.1.3.md) → Jira: FL-37
- [ ] **Task 4:** [Implement S3 contract document upload for supplier agreements.](./task-2.1.4.md) → Jira: FL-44
- [ ] **Task 5:** [Build Supplier Profile page (view, edit, documents tab).](./task-2.1.5.md) → Jira: FL-45

---

## 6. Edge Cases & Risk Mitigation
- **Duplicate supplier name**
  - *Mitigation:* System warns (does not block) if a submitted name closely matches an existing record using fuzzy matching. Returns a `{ warning, matches }` response for the PM to review before confirming creation.
- **Contract document upload failure (S3 timeout)**
  - *Mitigation:* Partial failure tolerance — if the S3 upload fails, the supplier profile is still saved without a document URL. The PM can re-attempt the upload from the Documents tab. A clear error toast explains the document was not saved.

---

## 7. Dependencies
- Blocked by Epic 1 (IAM) — RBAC middleware must exist.
- Blocked by Epic 1 Story 1.1 — `Users` table must exist for `Audit_Logs` FK.
- Unblocks Story 2.2 (Supplier-Product Price Mapping) — `Suppliers` table must exist before price mappings can be created.
- Unblocks Epic 4 (Smart Receiving) — receiving clerks must be able to select a verified supplier for each delivery.

---

## 8. PM Open Questions / Clarifications
- **userAskQuestion:** Should the system enforce that a contract document is uploaded before a supplier can be used in a receiving workflow, or is the document optional at onboarding and can be added later?
- **userAskQuestion:** Do we need a formal "Maker/Checker" approval workflow for new supplier registrations (Procurement Manager creates → Finance/Admin approves), or can authenticated Procurement Managers register suppliers autonomously?
