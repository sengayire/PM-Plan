# Task 2.1.5: Build Supplier Profile Page (View, Edit, Documents Tab)

## 1. Meta Information
- **Parent Story:** [User Story 2.1: Supplier Onboarding & Profile Management](./user-story-2.1.md)
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Jira:** FL-45
- **Estimate:** 8h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build the Supplier Profile page — the full detail view and management hub for a single supplier. This page hosts the profile overview, an edit form, a Documents tab for contract management, and suspend/activate controls. It is the primary UI surface accessed from the Supplier Directory.

## 3. Implementation Requirements

### 3.1. Page Route
- Route: `/suppliers/[id]` (Next.js App Router dynamic segment)
- Server component fetches initial supplier data via `GET /suppliers/:id`

### 3.2. Page Tabs
The page must have three tabs:

**Tab 1: Overview**
- Display all supplier fields: Name, Contact Person, Phone, Email, Location, Status badge, Created by, Created at
- If `is_active: false`, show a red "Suspended" banner with the `suspension_reason` displayed prominently

**Tab 2: Edit**
- Inline edit form pre-populated from supplier data
- Same validation rules as the Register Supplier form (FL-43)
- Save button triggers `PATCH /suppliers/:id`; success → invalidate React Query cache and switch back to Overview tab
- Suspend/Activate controls:
  - If Active: "Suspend Supplier" button (red, requires confirmation modal with mandatory reason textarea, min 10 chars)
  - If Suspended: "Re-activate Supplier" button (Admin only)

**Tab 3: Documents**
- If `contract_document_url` is null: "No contract attached" empty state with "Upload Contract" button
- If document exists: filename, upload date, "View Document" button (opens pre-signed URL in new tab via `GET /suppliers/:id/contract/view-url`)
- "Upload Contract" triggers the S3 upload flow (FL-44):
  1. File picker (PDF/JPG/PNG, max 10 MB)
  2. Progress indicator during upload
  3. On success: cache invalidation + document entry appears

### 3.3. Audit Trail Integration
- All profile edits and status changes write audit logs via the backend interceptor (FL-23)
- No frontend-specific audit logic needed

### 3.4. Breadcrumb Navigation
- `Suppliers > {Supplier Name}` breadcrumb at top of page

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Overview tab renders all supplier fields correctly
- [ ] Suspended supplier shows red banner with suspension reason
- [ ] Edit tab pre-populates form with current data; save triggers `PATCH` and returns to Overview
- [ ] Suspension modal requires non-empty reason (min 10 chars) before calling `PATCH /suppliers/:id/suspend`
- [ ] "Re-activate" button only visible to Admin role
- [ ] Documents tab shows empty state when no contract attached
- [ ] "View Document" button opens pre-signed URL in new tab
- [ ] File upload triggers S3 flow; progress indicator visible during upload
- [ ] Error toast shown if document upload fails; profile remains accessible

## 5. Dependencies
- **Requires:** FL-43 (Supplier REST API — all profile and suspend endpoints)
- **Requires:** FL-44 (S3 contract upload endpoints)
- **Requires:** FL-37 (Supplier Directory — links into this page)
- **Hooks into:** FL-46 (Scorecard tab — added in Sprint 4 post-Epic 4)
