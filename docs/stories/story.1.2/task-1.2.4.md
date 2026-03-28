# Task 1.2.4: Add CSV/PDF Export for Audit Logs

## 1. Meta Information
- **Parent Story:** [User Story 1.2: Immutable Audit Logging](./user-story-1.2.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Jira:** FL-25
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 4h

## 2. Objective
Add export capabilities to the Audit Log search interface, allowing Admins and Auditors to download filtered audit data as CSV or PDF for offline review, regulatory submissions, and FDA inspection packages.

## 3. Implementation Requirements

### 3.1. Backend Export Endpoint
- Add `GET /audit-logs/export?format=csv|pdf` accepting the same filter parameters as the search endpoint.
- **CSV:** Use a streaming response to handle large exports without timeout (use Node.js streams + `fast-csv` or built-in `csv-stringify`).
- **PDF:** Generate a formatted table report using `PDFKit` or `Puppeteer` (render the filtered data as a styled HTML table → PDF).
- Set response headers: `Content-Disposition: attachment; filename="audit-log-{date}.csv|pdf"`.
- Restrict to `Admin` and `Auditor` roles.

### 3.2. Frontend Integration
- Add "Export CSV" and "Export PDF" buttons to the filter panel header in the Audit Log UI.
- Buttons pass the current active filter state as query params to the export endpoint.
- Show a loading spinner on the button while the download is generating.
- Log the export action itself to `Audit_Logs` with `action: EXPORT`.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] "Export CSV" downloads a valid `.csv` file containing all columns matching the active filter.
- [ ] "Export PDF" downloads a formatted PDF report with the audit log data.
- [ ] Exporting 10,000 rows does not cause a server timeout (streaming response).
- [ ] Each export action is itself recorded in `Audit_Logs` (action: `EXPORT`, entity_type: `AuditLog`).
- [ ] Unauthorized roles receive `403 Forbidden` on the export endpoint.

## 5. Security & Dependencies
- Depends on Task 1.2.3 (Audit Log Search UI and backend endpoint).
- Export must use the same RBAC guard as the search endpoint.
