# Task 1.1.2: Implement JWT Authentication Service

## 1. Meta Information
- **Parent Story:** [User Story 1.1: Role-Based Access Control (RBAC) Setup](./user-story-1.1.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** [To be estimated by Engineering]

## 2. Objective
Develop the backend authentication service using JSON Web Tokens (JWT). This service will handle user login, verify credentials against the PostgreSQL `users` table, and generate a secure session token containing the user's operational role. This is critical to ensure that users (Admin, Manager, Warehouse Picker, Driver, Finance, QC Inspector) only access their authorized endpoints.

## 3. High-Level Requirements

### 3.1. Login Endpoint
- Create an API endpoint (e.g., `POST /auth/login`) designed to accept user credentials.
- Implement secure credential verification (e.g., using bcrypt to compare against the stored hash in the `users` table).
- Validate that the user account is active (`is_active = true`) before issuing a token.

### 3.2. JWT Generation
- Upon successful credential verification, generate a secure JWT signed with a localized secret key.
- **Required JWT Payload Claims:**
  - `sub`: The user's unique identifier (UUID).
  - `role`: The user's assigned role for downstream RBAC middleware.
  - `exp`: Expiration time (suggest 8-12 hours for a standard shift).

### 3.3. Standardized JSON Response
- The API must return a structured JSON response upon successful login containing the token and user role:
  ```json
  {
    "access_token": "<JWT_STRING>",
    "user": {
      "id": "<UUID>",
      "role": "<ROLE>"
    }
  }
  ```

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Login route (`POST /auth/login`) is implemented and correctly validates required payload fields.
- [ ] Unsuccessful logins (invalid password or email) return an appropriate `401 Unauthorized` response.
- [ ] Successful logins return the valid JWT and structured user data.
- [ ] JWT correctly includes the `role` and `sub` payload claims.
- [ ] Deactivated users are prevented from generating a token and receive `403 Forbidden` or `401 Unauthorized`.

## 5. Security & Compliance Context
- **Security Implications:** This component is the primary entry point to the system. The secret key used to sign the JWT must be highly secure and injected via environment variables.
- **Offline & Mobile Support:** Drivers and Warehouse Pickers will use these JWTs on mobile devices. Consider how tokens are cached securely on the frontend (`Auth.js` or secure storage) as per our offline-first constraint.

## 6. Open Questions / Clarifications
- *userAskQuestion*: Should we implement a Refresh Token strategy for mobile users (Drivers/Pickers) to prevent unexpected session expirations while in offline/low-connectivity environments, or is a long-lived JWT sufficient for the MVP Phase 1?
- *userAskQuestion*: Does Rwanda FDA compliance require us to log every successful and failed login attempt into the `Audit_Logs` table for platform security tracking?
