# Task 2.3.3: Build Scorecard Alert Badges and Supplier Directory Risk Flags

## 1. Meta Information
- **Parent Story:** [User Story 2.3: Supplier Performance Scorecard](./user-story-2.3.md)
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Jira:** FL-47
- **Estimate:** 4h
- **Sprint:** Sprint 4+ (blocked on FL-46 — real scorecard data required)
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Add risk alert visualisation to the Supplier Directory and Supplier Profile scorecard widget — highlighting suppliers whose QC rejection rate exceeds the 20% threshold with a red indicator and a risk badge in the directory list.

## 3. Implementation Requirements

### 3.1. Alert Threshold
- **Trigger:** `qcRejectionRate > 20` in the last 30-day period.
- Threshold is evaluated at display time from the scorecard API response — no separate alert endpoint needed.

### 3.2. Scorecard Widget — Metric Highlighting
In the Supplier Profile scorecard widget (built in FL-39):
- When `qcRejectionRate > 20`: render the rejection rate metric value in **red text** with a warning icon (⚠).
- Tooltip on hover: "This supplier's rejection rate exceeds the 20% risk threshold (last 30 days)."
- Other metrics remain styled normally.

### 3.3. Supplier Directory Risk Badge
In the Supplier Directory table (built in FL-37):
- The "Risk" column slot (reserved in FL-37) is activated here.
- When a supplier's `qcRejectionRate > 20`: render a red `HIGH RISK` chip in the Risk column.
- When `qcRejectionRate ≤ 20` or `hasData: false`: render nothing (empty cell).
- Risk column data sourced from a bulk scorecard endpoint: `GET /suppliers/scorecards/summary?period=30d` that returns `[{ supplierId, qcRejectionRate, isHighRisk }]` for all suppliers. This avoids N+1 requests from the directory page.

### 3.4. Bulk Summary Endpoint (Backend)
- Route: `GET /suppliers/scorecards/summary`
- Query param: `?period=30d` (default)
- Returns lightweight array:
```json
[
  { "supplierId": "...", "qcRejectionRate": 23.5, "isHighRisk": true },
  { "supplierId": "...", "qcRejectionRate": 5.1, "isHighRisk": false }
]
```
- Only includes suppliers with `hasData: true`; omits suppliers with no data.
- Cached in Redis with key `scorecard_summary:30d` and TTL of 5 minutes.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Scorecard widget renders rejection rate in red with warning icon when > 20%
- [ ] Tooltip appears on hover of red rejection rate metric
- [ ] `HIGH RISK` badge appears in Supplier Directory Risk column for suppliers > 20% rejection rate
- [ ] Risk column is empty (no badge) for suppliers with rejection rate ≤ 20% or no data
- [ ] `GET /suppliers/scorecards/summary` returns lightweight array (no N+1 on directory page)
- [ ] Risk data updates within the Redis TTL window (5 minutes) after a new QC_Log insert

## 5. Dependencies
- **Requires:** FL-46 (real aggregation service — needs `qcRejectionRate` in scorecard response)
- **Requires:** FL-37 (Supplier Directory — "Risk" column slot reserved and ready)
- **Requires:** FL-39 (Supplier Profile scorecard widget — metric highlighting applied here)
