# Task 1.1.5: Frontend Admin UI for User Management

## 1. Meta Information
- **Parent Story:** [User Story 1.1: Role-Based Access Control (RBAC) Setup](./user-story-1.1.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** [To be estimated by Engineering]

## 2. Objective
Develop the front-end user interface within the Admin Dashboard to read, provision, update, and deactivate system user accounts. This UI is the primary tool for System Administrators to manage the platform's workforce and assign the appropriate access roles necessary for the "Digital Twin" operations (Pickers, Drivers, Managers, etc.).

## 3. High-Level Requirements

### 3.1. User Directory Dashboard (Read)
- Build a structured data table to display all current users in the system.
- **Required Columns:**
  - Name
  - Email / Username
  - Assigned Role (e.g., `Driver`, `Warehouse Picker`, `Admin`)
  - Status (Active / Inactive)
  - Date Added
- Include basic filtering (e.g., filter by Role or Status) and a search bar to quickly locate users by name or email.
- **Real-Time Status Updates:** Integrate a WebSocket or Server-Sent Events (SSE) listener. When a newly provisioned user successfully activates their account, the frontend table must automatically update their Status to 'Active' in real-time, without requiring the Admin to refresh the page.

### 3.2. User Creation Form (Create)
- Build a form interface to provision new user accounts.
- **Form Fields:** First Name, Last Name, Email, Phone Number, and Role Assignment (Dropdown).
- **Delivery Method Toggle:** Include a toggle or checkbox group allowing the Admin to explicitly choose how the activation link is sent (Email, SMS, or Both).
- Upon submission, trigger a `POST` request to the backend API (`/users`).
- Handle frontend validation (e.g., required fields, valid email format) before submission.

### 3.3. User Modification (Update & Soft Delete)
- Implement an edit interface to adjust a user's details or change their role.
- **Role Constraint:** Changing a user's role should trigger a warning module informing the Admin that the user's active session will be invalidated, forcing them to re-login. (Backend handles the actual invalidation, UI provides the warning context).
- **Deactivation (Soft Delete):** 
  - Provide a highly visible "Deactivate User" action. 
- **Resend Activation Link:** Provide an action button (restricted to Admins) for users currently in the `is_active: false` state to trigger the backend to regenerate and resend their 24-hour activation link via their preferred delivery method.
  - **CRITICAL:** Do *not* label this button "Delete". It must execute a `PATCH` request setting `is_active: false` on the backend. This enforces the immutable ledger requirement for Rwanda FDA compliance.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] User Directory renders a paginated or scrollable list of users fetched from the API.
- [ ] Admins can successfully create a new user account with a designated role.
- [ ] Form validations prevent empty critical fields or malformed emails.
- [ ] Admins can change a user's role (with UI warning about session invalidation).
- [ ] Admins can deactivate a user (translating to `is_active: false`), and the UI visually reflects the inactive status (e.g., greyed out).
- [ ] Standard users (e.g., Drivers, Managers) cannot access this dashboard. The route must be protected by the `Auth.js` session role.

## 5. Security & Architecture Context
- **Route Guarding:** This entire UI module must reside behind a Next.js layout or route guard that strictly checks `session.user.role === 'Admin'`. Unauthorized attempts should redirect to an informational "Access Denied" view, not immediately log the user out.

## 6. Open Questions / Clarifications
- *userAskQuestion*: Should the system automatically generate strong temporary passwords and email them to new users upon creation, or will the Admin manually communicate credentials off-platform for the Phase 1 MVP?
- *userAskQuestion*: Do we require a "Privilege Escalation Warning" (e.g., a secondary confirmation modal or OTP prompt) explicitly when an Admin attempts to grant *another* user the `Admin` role?
