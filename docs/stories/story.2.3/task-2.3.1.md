# Task 2.3.1: Supplier Scorecard Stub API + Frontend Widget (Sprint 2 Placeholder)

## 1. Meta Information
- **Parent Story:** [User Story 2.3: Supplier Performance Scorecard](./user-story-2.3.md)
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Jira:** FL-39
- **Estimate:** 4h
- **Sprint:** Sprint 2 (placeholder — real data in Sprint 4)
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build a stub scorecard API endpoint and a frontend widget on the Supplier Profile page that renders the correct UI shell for Sprint 2. The widget returns hardcoded "No data yet" state now and will be wired to real aggregation data in FL-46 (Sprint 4+) with no layout changes required.

## 3. Implementation Requirements

### 3.1. Backend Stub Endpoint
- Route: `GET /suppliers/:id/scorecard`
- Always returns the "no data" response structure in Sprint 2:
```json
{
  "supplierId": "...",
  "period": "30d",
  "hasData": false,
  "metrics": {
    "totalDeliveries": null,
    "qcRejectionRate": null,
    "avgLeadTimeDays": null,
    "onTimeDeliveryRate": null
  },
  "lastCalculatedAt": null
}
```
- Accept optional query param `?period=30d|90d|all` — stub ignores value but must accept it to match the real endpoint signature.
- Return `404` if supplier `id` does not exist.

### 3.2. Frontend Widget
- Location: Tab 4 "Performance" on the Supplier Profile page (`/suppliers/[id]`)
- When `hasData: false`: render the empty state card:
  > "Scorecard data will appear after the first delivery is logged."
  > (Subtitle: "Data is calculated from Receiving Logs and QC records.")
- Period selector (30d / 90d / All time) should render but be disabled with a tooltip: "Available after first delivery."
- Widget must render without errors regardless of the stub response.

### 3.3. Contract (for FL-46 handoff)
This stub defines the full response contract that FL-46 will implement. The frontend widget must be built against this contract — no structural changes to the widget will be needed in Sprint 4, only the backend response data will change.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `GET /suppliers/:id/scorecard` returns the no-data stub response
- [ ] `GET /suppliers/:id/scorecard` returns `404` for unknown supplier ID
- [ ] Widget renders "No data yet" empty state on the Supplier Profile page
- [ ] Period selector renders but is disabled with tooltip
- [ ] No console errors when widget mounts with stub data

## 5. Dependencies
- **Requires:** FL-43 (Supplier REST API — supplier must exist)
- **Requires:** FL-45 (Supplier Profile page — Tab 4 slot)
- **Unblocks:** FL-46 (real aggregation service — Sprint 4)
