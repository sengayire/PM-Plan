# Task 2.3.2: Implement Real-Time Scorecard Aggregation Service (Post-Epic 4)

## 1. Meta Information
- **Parent Story:** [User Story 2.3: Supplier Performance Scorecard](./user-story-2.3.md)
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Jira:** FL-46
- **Estimate:** 8h
- **Sprint:** Sprint 4+ (blocked on Epic 4 — Receiving Logs must exist)
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Replace the stub scorecard endpoint (FL-39) with a real aggregation service that computes supplier performance metrics from `Receiving_Logs` and `QC_Logs`. This task is the backend engine powering the scorecard widget.

## 3. Implementation Requirements

### 3.1. Data Sources (Epic 4 schema dependencies)
Required tables from Epic 4:
- `Receiving_Logs`: `supplier_id`, `received_at`, `expected_delivery_date`, `status` (RECEIVED/REJECTED)
- `QC_Logs`: `receiving_log_id`, `supplier_id`, `qc_result` (PASSED/FAILED/PARTIAL), `rejected_weight_kg`, `total_weight_kg`

If these tables do not yet exist, this task is blocked. Do not mock them — verify Epic 4 schema is deployed first.

### 3.2. Metrics Calculation

| Metric | Formula |
|--------|---------|
| Total Deliveries | COUNT(`Receiving_Logs`) for supplier in period |
| QC Rejection Rate (%) | SUM(rejected_weight_kg) / SUM(total_weight_kg) × 100 |
| Avg Lead Time (days) | AVG(`received_at` - `expected_delivery_date`) |
| On-Time Delivery Rate (%) | COUNT(received_at ≤ expected_delivery_date) / COUNT(*) × 100 |

### 3.3. Period Filter
- `?period=30d` — last 30 days from `NOW()`
- `?period=90d` — last 90 days
- `?period=all` — all time (no date filter)

### 3.4. Response Format (matches FL-39 stub contract)
```json
{
  "supplierId": "...",
  "period": "30d",
  "hasData": true,
  "deliveryCount": 12,
  "confidence": "high",
  "metrics": {
    "totalDeliveries": 12,
    "qcRejectionRate": 8.3,
    "avgLeadTimeDays": 1.5,
    "onTimeDeliveryRate": 91.7
  },
  "lastCalculatedAt": "2026-03-28T10:00:00Z"
}
```

### 3.5. Low-Confidence Indicator
- If `totalDeliveries < 3`, set `"confidence": "low"` in the response.
- Frontend widget (already built in FL-39) will display "(Based on N deliveries)" subtitle when `confidence: "low"`.

### 3.6. Cache Strategy
- Cache scorecard results in Redis with key `scorecard:{supplierId}:{period}` and TTL of 5 minutes.
- Cache must be **invalidated** on any INSERT or UPDATE to `Receiving_Logs` or `QC_Logs` for the affected `supplier_id`.
- Use the NestJS event system or a database trigger + queue to detect changes and invalidate.

### 3.7. No-Data State
- If supplier has zero receiving events, return `hasData: false` with null metrics (same as stub) — this ensures backward compatibility with the frontend widget.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `GET /suppliers/:id/scorecard?period=30d` returns real computed metrics from DB
- [ ] QC rejection rate calculated correctly (weight-based, not count-based)
- [ ] `confidence: "low"` returned when fewer than 3 deliveries in period
- [ ] Cache hit returns same result within TTL; invalidated after Receiving_Log update for that supplier
- [ ] `hasData: false` returned when supplier has no receiving history
- [ ] Unit tests cover: metric formulas, low-confidence threshold, cache invalidation trigger
- [ ] Integration test: create 2 receiving events → scorecard reflects data

## 5. Dependencies
- **Requires:** FL-39 (stub endpoint — must be replaced, not deleted; keep route signature identical)
- **Requires:** Epic 4 schema: `Receiving_Logs` and `QC_Logs` tables with `supplier_id` FK
- **Unblocks:** FL-47 (alert badges — needs real data to trigger thresholds)
