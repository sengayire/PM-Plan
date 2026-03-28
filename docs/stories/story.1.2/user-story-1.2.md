# User Story 1.2: Immutable Audit Logging

## 1. Meta Information
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Story Priority:** High (MUST — Rwanda FDA compliance requirement)
- **Story Point Estimation:** 19h (4 tasks)
- **Status:** To Do
- **Jira:** FL-14

## 2. User Story
**As a** System Auditor,
**I want** a non-editable log of all critical data changes across the platform,
**So that** I can trace discrepancies and satisfy Rwanda FDA inspection requirements at any time.

---

## 3. Business Context
Rwanda FDA compliance mandates that AgriFlow maintain a complete, tamper-proof record of every inventory mutation, user action, and financial transaction. Without an immutable audit trail, the platform cannot pass regulatory inspections. This also serves as an internal accountability mechanism — any discrepancy in stock levels can be traced back to a specific user action with full before/after context.

---

## 4. Acceptance Criteria (BDD Format)

### Scenario 1: Inventory change triggers audit entry
**Given** any authenticated user performs a `POST`, `PATCH`, or `DELETE` on an inventory entity
**When** the request completes (success or failure)
**Then** the system must write an entry to `Audit_Logs` containing: `user_id`, `timestamp`, `entity_type`, `entity_id`, `action`, `old_value`, and `new_value`
**And** this entry must be immutable — no `UPDATE` or `DELETE` is permitted on the `Audit_Logs` table.

### Scenario 2: Auditor searches and exports logs
**Given** a System Auditor navigates to the Audit Log search interface
**When** they filter by date range, user, entity type, and action
**Then** results must be returned in under 2 seconds for queries spanning up to 90 days of data
**And** the Auditor must be able to export the filtered results as CSV or PDF.

---

## 5. Technical Tasks (Developer Implementation)
- [ ] **Task 1:** [Create `Audit_Logs` table with indexed columns.](./task-1.2.1.md)
- [ ] **Task 2:** [Implement global NestJS interceptor to log POST/PATCH/DELETE requests.](./task-1.2.2.md)
- [ ] **Task 3:** [Build Audit Log search UI with filters (date range, user, entity, action).](./task-1.2.3.md)
- [ ] **Task 4:** [Add CSV/PDF export for audit log query results.](./task-1.2.4.md)

---

## 6. Edge Cases & Risk Mitigation
- **Bulk import triggers thousands of log entries simultaneously**
  - *Mitigation:* Use batch inserts with an async queue (e.g., BullMQ) to write audit entries in the background and prevent API response latency from spiking.
- **Audit log table grows beyond query performance thresholds**
  - *Mitigation:* Implement PostgreSQL date-based table partitioning (monthly partitions) from day one so the query planner scans only the relevant partition.

---

## 7. Dependencies
- Depends on Story 1.1 (RBAC Setup) — `user_id` foreign key in `Audit_Logs` references the `Users` table.
- Blocks Epic 4 (Smart Receiving) and Epic 5 (Inventory) — those modules rely on audit entries for FDA traceability.

---

## 8. PM Open Questions / Clarifications
- **userAskQuestion:** Should failed API requests (e.g., a `403 Forbidden`) also be logged in `Audit_Logs`, or only successful mutations?
- **userAskQuestion:** Is there a data retention policy — e.g., must audit logs be kept for a minimum of X years per Rwanda FDA regulations?
