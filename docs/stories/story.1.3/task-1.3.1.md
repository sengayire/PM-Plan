# Task 1.3.1: Implement Password Reset Endpoint

## 1. Meta Information
- **Parent Story:** [User Story 1.3: Password Reset & Session Management](./user-story-1.3.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Jira:** FL-26
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 4h

## 2. Objective
Implement the backend password reset flow — from initial reset request through secure token generation to final password update. The flow must be secure against account enumeration, brute-force attacks, and token reuse.

## 3. Implementation Requirements

### 3.1. Reset Request Endpoint (`POST /auth/reset-password/request`)
- Accept `{ email: string }` in the request body.
- Look up the email in the `Users` table.
- **Regardless of whether the email exists**, return a generic `200 OK` response (`"If this email is registered, you will receive a reset link"`) to prevent account enumeration.
- If email exists and `is_active: true`: generate a cryptographically secure random token (32 bytes, hex-encoded), store it hashed in a `password_reset_tokens` table with a 15-minute `expires_at` timestamp, and dispatch the reset email via the NestJS mailer service.
- **Rate limit:** Maximum 3 requests per email per hour. Return `429 Too Many Requests` on excess.

### 3.2. Reset Token Validation + Password Update (`POST /auth/reset-password/confirm`)
- Accept `{ token: string, new_password: string }`.
- Look up the token hash in `password_reset_tokens`.
- Reject if: token not found, already used (`used_at IS NOT NULL`), or expired (`expires_at < NOW()`).
- On valid token: hash the new password with bcrypt, update `Users.password_hash`, mark the token as used (`used_at = NOW()`), and return `200 OK`.
- Invalidate all active JWTs for this user on successful reset.

### 3.3. Supporting Schema
- Create `password_reset_tokens` table: `id` (UUID), `user_id` (FK), `token_hash` (VARCHAR, UNIQUE), `expires_at` (TIMESTAMP), `used_at` (TIMESTAMP, nullable), `created_at`.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `POST /auth/reset-password/request` returns `200` for both registered and unregistered emails.
- [ ] A reset token is generated and emailed only when the email belongs to an active user.
- [ ] Token expires after 15 minutes — a request with an expired token returns `400 Bad Request`.
- [ ] Token is single-use — a second confirmation request with the same token returns `400`.
- [ ] 4th reset request within 1 hour returns `429 Too Many Requests`.
- [ ] Successful reset is logged to `Audit_Logs`.

## 5. Security & Dependencies
- Depends on Task 1.1.2 (JWT Auth Service) — JWT invalidation on reset.
- Depends on Task 1.2.2 (Audit Logging interceptor).
- Depends on Task 1.1.6 (Credential Dispatch) for sending the reset email.
