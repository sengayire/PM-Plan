# Task 2.2.3: Build Supplier Directory UI (Next.js)

## 1. Meta Information
- **Parent Story:** [User Story 2.2: Supplier Registration & Product Price Book](./user-story-2.2.md)
- **Epic:** Epic 2: Partner & Supplier Ecosystem (FL-5)
- **Jira:** FL-37
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 6h

## 2. Objective
Build the Supplier Directory — the primary frontend page where Buyers and Procurement Managers can view, search, and register suppliers.

## 3. High-Level Requirements

### 3.1. Supplier List Page (`/suppliers`)
- Data table with columns: Name, Location, Contact Number, Status (Active/Inactive), Actions.
- Debounced search input (500ms) filtering by supplier name.
- "Active Only" toggle to hide inactive suppliers.
- Pagination controls.
- Clicking a row navigates to the Supplier Profile page (`/suppliers/:id`).
- Inactive suppliers shown with faded styling + "Inactive" badge.
- "Register New Supplier" primary CTA button.

### 3.2. Register New Supplier Form
- Modal or dedicated route `/suppliers/new`.
- Fields: Name (required), Contact Number, Location, Bank Details (optional).
- `react-hook-form` + Zod validation. Required fields validated client-side.
- On success (`201`): close modal, refresh supplier list, show success toast.
- On API error: display error inline.

### 3.3. Auth & RBAC
- Route guarded by Auth.js — unauthenticated users redirected to login.
- "Register New Supplier" button and form visible only to `Buyer`, `Procurement Manager`, `Admin` roles.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Supplier list renders with pagination from `GET /suppliers`.
- [ ] Search input filters results within 500ms of the user stopping typing.
- [ ] Submitting the register form with missing required fields shows validation errors without calling the API.
- [ ] Successfully registering a supplier adds them to the list without a full page reload.
- [ ] A Picker/Driver role cannot see the "Register New Supplier" button.
