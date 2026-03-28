# User Story 2.3: Supplier Performance Scorecard

## 1. Meta Information
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Story Priority:** Medium (SHOULD — important for procurement decisions, not blocking MVP)
- **Story Point Estimation:** ~16h (3 tasks)
- **Status:** To Do
- **Jira:** FL-41
- **Sprint Dependency:** Depends on Epic 4 (Smart Receiving) data — fully deliverable in Sprint 4+

## 2. User Story
**As a** Procurement Manager,
**I want to** view a real-time performance scorecard for each supplier showing their QC rejection rate, delivery lead time, and on-time delivery rate,
**So that** I can make data-driven sourcing decisions, renegotiate with underperforming suppliers, and give farmers transparent, objective feedback on their produce quality.

---

## 3. Business Context
AgriFlow's Farmer/Supplier persona explicitly needs "performance-based ratings" (GEMINI.md Section 3). Without objective scorecard data, sourcing decisions rely on gut feel, and there is no mechanism to identify which suppliers are driving the most produce loss through poor quality or late deliveries. This story transforms the supplier profile into an actionable performance management tool.

---

## 4. Acceptance Criteria (BDD Format)

### Scenario 1: Viewing a supplier's live scorecard
**Given** a supplier has at least one completed receiving event in `Receiving_Logs`
**When** the Procurement Manager opens the Supplier Profile
**Then** the Scorecard widget displays: Total Deliveries, QC Rejection Rate (%), Average Lead Time (days), and On-Time Delivery Rate (%)
**And** all metrics are calculated from actual `Receiving_Logs` and `QC_Logs` data with a time period selector (30d / 90d / All time).

### Scenario 2: High-risk supplier alert
**Given** a supplier's QC rejection rate exceeds 20% in the last 30 days
**When** the Procurement Manager views the Scorecard or Supplier Directory
**Then** the rejection rate metric is highlighted in red
**And** a risk alert badge appears on the supplier's row in the Supplier Directory list.

### Scenario 3: No data state
**Given** a supplier has no receiving history yet
**When** the Scorecard is rendered
**Then** all metrics display a "No data yet" state with a message: "Scorecard data will appear after the first delivery is logged."

---

## 5. Technical Tasks (Developer Implementation)
- [ ] **Task 1:** [Supplier Scorecard stub API + frontend widget (Sprint 2 placeholder).](./task-2.3.1.md) → Jira: FL-39
- [ ] **Task 2:** [Implement real-time scorecard aggregation service (post-Epic 4).](./task-2.3.2.md) → Jira: FL-46
- [ ] **Task 3:** [Build scorecard alert badges and supplier directory risk flags.](./task-2.3.3.md) → Jira: FL-47

---

## 6. Edge Cases & Risk Mitigation
- **Supplier has only 1-2 deliveries**
  - *Mitigation:* Display a low-confidence indicator ("Based on N deliveries") for suppliers with fewer than 3 receiving events to prevent misleading conclusions.
- **Backdated or corrected receiving logs**
  - *Mitigation:* Scorecard metrics must recalculate automatically when `Receiving_Logs` are amended (e.g., QC dispute resolution). Scorecard cache must be invalidated on any insert or update to `Receiving_Logs` for this supplier.

---

## 7. Dependencies
- Blocked by Epic 4 (Smart Receiving) — `Receiving_Logs` and `QC_Logs` tables with `supplier_id` FK must exist.
- Sprint 2: Deliverable as a stub/placeholder widget (FL-39).
- Sprint 4+: Fully deliverable with real data once Epic 4 is deployed.

---

## 8. PM Open Questions / Clarifications
- **userAskQuestion:** Should the 20% QC rejection threshold trigger an automated notification (email/SMS) to the Procurement Manager, or is visual flagging on the UI sufficient for MVP?
- **userAskQuestion:** Should suppliers be able to see their own scorecard (supplier-facing portal), or is this strictly an internal AgriFlow tool for Phase 1?
