# Task 1.3.3: JWT Refresh Token Rotation & Session Expiry

## 1. Meta Information
- **Parent Story:** [User Story 1.3: Password Reset & Session Management](./user-story-1.3.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Jira:** FL-28
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 4h

## 2. Objective
Implement JWT refresh token rotation on the backend and configure the Auth.js frontend session to transparently refresh access tokens before expiry. This is critical for warehouse pickers and drivers on mobile devices who need uninterrupted sessions during long shifts.

## 3. Implementation Requirements

### 3.1. Backend: Refresh Token Endpoint
- Add a `refresh_tokens` table: `id` (UUID), `user_id` (FK), `token_hash` (VARCHAR, UNIQUE), `expires_at` (TIMESTAMP, 7-day TTL), `used_at` (TIMESTAMP, nullable), `created_at`.
- Implement `POST /auth/refresh` accepting `{ refresh_token: string }` in a secure `httpOnly` cookie or request body.
- **Rotation logic:** On each valid refresh request, invalidate the old refresh token (`used_at = NOW()`) and issue a **new** access token (8–12h) + a **new** refresh token (7 days). This is "refresh token rotation" — a stolen token can only be used once.
- Return `401` for expired, used, or unrecognized refresh tokens.

### 3.2. Frontend: Auth.js Session Configuration
- Configure the Auth.js `jwt` callback to embed the refresh token and access token expiry in the session.
- In the `session` callback, check if the access token is within 5 minutes of expiry; if so, call `POST /auth/refresh` transparently and update the session with the new tokens.
- If the refresh call fails (expired refresh token), clear the session and redirect to login.

### 3.3. Mobile Consideration
- Document in a code comment the offline strategy: mobile apps (Flutter/React Native) should persist the refresh token in secure device storage and call the refresh endpoint on app resume if the access token has expired.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `POST /auth/refresh` with a valid refresh token returns a new access token and new refresh token.
- [ ] The old refresh token is marked `used_at` and cannot be reused — attempting reuse returns `401`.
- [ ] Auth.js frontend session auto-refreshes without forcing the user to log in again mid-shift.
- [ ] An expired refresh token (>7 days) returns `401` and the frontend redirects to login.
- [ ] Refresh token rotation is logged to `Audit_Logs`.

## 5. Security & Dependencies
- Depends on Task 1.1.2 (JWT Auth Service) for initial token issuance.
- Depends on Task 1.1.4 (Auth.js frontend) for session callback configuration.
- Depends on Task 1.2.2 (Audit Logging) for logging refresh events.
