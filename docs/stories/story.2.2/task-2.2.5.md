# Task 2.2.5: Implement Supplier Scorecard API (Stub + Widget)

## 1. Meta Information
- **Parent Story:** [User Story 2.2: Supplier Registration & Product Price Book](./user-story-2.2.md)
- **Epic:** Epic 2: Partner & Supplier Ecosystem (FL-5)
- **Jira:** FL-39
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 4h

## 2. Objective
Implement the Supplier Scorecard endpoint and frontend widget as a stub in Sprint 2, to be populated with real data once Epic 4 (Smart Receiving) delivers the `Receiving_Logs` and `QC_Logs` tables. This establishes the API contract and UI component early so Sprint 4 can simply wire up live data without frontend changes.

## 3. Implementation Requirements

### 3.1. Backend: Scorecard Endpoint
- `GET /suppliers/:id/scorecard`
- **Sprint 2 (stub):** Return a fixed placeholder response structure:
  ```json
  {
    "total_deliveries": 0,
    "qc_rejection_rate_pct": 0,
    "avg_lead_time_days": 0,
    "on_time_delivery_rate_pct": 0,
    "data_available": false
  }
  ```
- **Sprint 4 (live):** Replace stub logic with real aggregation queries against `Receiving_Logs` and `QC_Logs`.
- Restrict to `Admin`, `Procurement Manager`, `Buyer` roles.

### 3.2. Frontend: Scorecard Widget
- Add a "Scorecard" section to the Supplier Profile page (below the details header).
- Display 4 KPI cards: Total Deliveries, QC Rejection Rate, Avg Lead Time, On-Time Rate.
- When `data_available: false`: show a "No delivery data yet" placeholder state for each KPI.
- When `data_available: true`: render the actual metric values with appropriate formatting.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `GET /suppliers/:id/scorecard` returns the stub response structure in Sprint 2.
- [ ] The Scorecard widget renders on the Supplier Profile page without errors.
- [ ] "No delivery data yet" placeholder state displays correctly when `data_available: false`.
- [ ] The API contract (response shape) is documented in a code comment for Epic 4 implementation reference.

## 5. Dependencies
- Sprint 2: No dependencies beyond `Suppliers` table existing.
- Sprint 4: Depends on `Receiving_Logs` (Epic 4) and `QC_Logs` (Epic 4) tables to supply real metrics.
