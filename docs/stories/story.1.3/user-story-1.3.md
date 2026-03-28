# User Story 1.3: Password Reset & Session Management

## 1. Meta Information
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Story Priority:** High (MUST)
- **Story Point Estimation:** 12h (3 tasks)
- **Status:** To Do
- **Jira:** FL-15

## 2. User Story
**As a** platform user (Admin, Manager, Picker, Driver, or Finance Officer),
**I want to** securely reset my password and have my session managed reliably,
**So that** I can regain access without admin intervention and the system automatically protects me from unauthorized session re-use.

---

## 3. Business Context
Warehouse pickers and drivers operate on mobile devices in low-connectivity environments. A broken password reset flow or sudden session expiry mid-shift causes operational disruption and forces manual admin intervention. This story ensures users can self-serve access recovery while keeping the platform secure from token abuse.

---

## 4. Acceptance Criteria (BDD Format)

### Scenario 1: Secure password reset flow
**Given** a user requests a password reset by submitting their email address
**When** the email is submitted (regardless of whether it is registered)
**Then** a time-limited (15-minute) secure reset link is sent to registered accounts
**And** the UI must not indicate whether the email address exists in the system (prevent account enumeration attacks).

### Scenario 2: Session expiry redirects to login
**Given** a user's session JWT has expired
**When** they make any authenticated API request
**Then** the system returns `401 Unauthorized`
**And** the frontend automatically redirects the user to the login page without data loss where possible.

### Scenario 3: Rate limiting on reset requests
**Given** a user has already submitted 3 password reset requests within 1 hour
**When** they attempt a 4th request
**Then** the system must reject it with a rate-limit error and inform the user to wait before retrying.

---

## 5. Technical Tasks (Developer Implementation)
- [ ] **Task 1:** [Implement password reset endpoint with secure token generation and expiry.](./task-1.3.1.md)
- [ ] **Task 2:** [Build password reset frontend flow (request, token validation, new password).](./task-1.3.2.md)
- [ ] **Task 3:** [Add JWT refresh token rotation and session expiry handling.](./task-1.3.3.md)

---

## 6. Edge Cases & Risk Mitigation
- **Multiple reset requests in quick succession**
  - *Mitigation:* Rate-limit to 3 reset attempts per hour per email address at the API layer. Return a generic "too many requests" response without revealing account existence.
- **Reset link used after expiry**
  - *Mitigation:* Tokens are single-use with a 15-minute TTL stored in the DB. On expiry, reject the token with a clear message and offer a "Resend reset email" CTA.

---

## 7. Dependencies
- Depends on Story 1.1 (RBAC Setup) — the `Users` table and JWT infrastructure must exist.
- Depends on Story 1.2 (Audit Logging) — password reset and token rotation events must be logged.

---

## 8. PM Open Questions / Clarifications
- **userAskQuestion:** For mobile users (Pickers/Drivers) using offline-first apps, should we implement a longer-lived refresh token strategy to avoid unexpected session expiry during connectivity gaps, or is a 12-hour access token sufficient for Sprint 1?
- **userAskQuestion:** Should password reset notifications (success/failure) be logged in `Audit_Logs` for security tracking?
