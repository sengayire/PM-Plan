# Task 2.1.3: Build Supplier Directory List UI (Next.js)

## 1. Meta Information
- **Parent Story:** [User Story 2.1: Supplier Onboarding & Profile Management](./user-story-2.1.md)
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Jira:** FL-37
- **Estimate:** 6h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build the Supplier Directory page — a searchable, filterable list of all registered suppliers. This is the primary entry point for the Procurement Manager to find, view, and manage supplier relationships.

## 3. Implementation Requirements

### 3.1. Page Route
- Route: `/suppliers` (Next.js App Router page)
- Server component for initial SSR fetch; React Query for client-side pagination and search.

### 3.2. Data Table
- Display columns: **Name**, **Contact Person**, **Location**, **Status** (Active / Suspended badge), **Contract** (icon if document attached), **Actions** (View Profile button)
- Status badge: green chip for Active, red chip for Suspended
- Contract icon: paperclip/document icon if `contract_document_url` is not null

### 3.3. Search & Filters
- Debounced search input (300ms) hitting `GET /suppliers?q={term}`
- Toggle filter: "Show suspended" checkbox (hidden by default; only visible to Admin role)
- URL query params must update on filter change (support browser back/forward navigation)

### 3.4. Empty & Loading States
- Empty state: illustration + "No suppliers registered yet. Click 'Register Supplier' to get started."
- Loading state: skeleton rows (not spinner) for perceived performance

### 3.5. Register Supplier CTA
- "Register Supplier" button in page header — navigates to `/suppliers/new`
- Button visible only to Procurement Manager and Admin roles (hide for read-only roles)

### 3.6. Risk Alert Badge Integration (Sprint 4 hook)
- Reserve a column slot labelled "Risk" — render empty placeholder for now.
- This slot will be populated by FL-47 (scorecard alert badges) in Sprint 4 without layout changes.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Page loads and displays supplier list with correct columns
- [ ] Search input debounces and triggers API call with `?q=` param
- [ ] Active/Suspended badge renders correctly based on `is_active` value
- [ ] Contract icon appears when `contract_document_url` is non-null
- [ ] "Show suspended" toggle is hidden from non-Admin roles
- [ ] "Register Supplier" button hidden from read-only roles
- [ ] Empty state renders when no suppliers exist
- [ ] Skeleton loading state renders while data is fetching

## 5. Dependencies
- **Requires:** FL-43 (Supplier REST API) — `GET /suppliers` endpoint
- **Requires:** FL-19 (Auth.js frontend — role context for UI guards)
- **Unblocks:** FL-45 (Supplier Profile page — links to profile from directory rows)
- **Hooks into:** FL-47 (scorecard risk badges — Sprint 4 addition)
