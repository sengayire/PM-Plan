# Task 1.1.6: Automated User Onboarding & Credential Dispatch (NestJS)

## 1. Meta Information
- **Parent Story:** [User Story 1.1: Role-Based Access Control (RBAC) Setup](./user-story-1.1.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
When an Admin provisions a new user, they will toggle the preferred delivery method (Email, SMS, or Both). The system will dispatch a notification containing a secure activation link. The newly created account will default to an inactive status (`is_active: false`). The user will click the link, be presented with a "Create Password" secure page, and upon submission, the system will seamlessly transition their database status to `is_active: true`.

## 3. High-Level Requirements

### 3.1. User Creation State Machine Update
- Modify the `POST /users` endpoint (Task 1.1.5) to enforce `is_active: false` by default for all new system users.
- Generate a secure, cryptographically hashed "Activation Token" bound to the user's ID.

### 3.2. Notification Dispatch Service
- Develop an asynchronous Notification Dispatch Service in the NestJS application.
- Trigger the service via a fast event emitter immediately after a user is committed to the database.
- The service will construct a personalized payload containing a "Magic Link" (e.g., `https://agriflow-domain.com/activate?token=XYZ...`).

### 3.3. First-Time Activation Endpoint
- Create a specific `POST /auth/activate` endpoint.
- This endpoint must verify the strictly 24-hour activation token payload and accept the user's newly defined secure password.
- Upon successful execution, it sets the user record to `is_active: true`, securely hashes the new password to the database, invalidates the activation token, and returns the standard session JWT.
- **Real-time Event:** Emit a `{ userId, status: 'Active' }` WebSocket event upon successful activation to instantly notify connected Admin clients.
- Create a `POST /users/:id/resend-activation` endpoint (Admin-only) to successfully regenerate the 24-hour token and resend the payload.

## 4. Architecture & Technology Identification
To execute this reliably within our chosen **NestJS** backend framework, developers should integrate the following technologies:

| Component | Recommended Technology | Purpose & Justification |
| :--- | :--- | :--- |
| **Email Transport Engine** | `@nestjs-modules/mailer` | NestJS wrapper for `Nodemailer`. Integrates perfectly with the Nest Dependency Injection container. |
| **Email Delivery API** | **Resend** or **SendGrid** | Third-party managed email providers to guarantee high deliverability and avoid landing in spam filters during MVP rollout. |
| **SMS Gateway API** | **Africa's Talking** SDK | The de-facto, highly reliable standard for SMS aggregators in Rwanda and East Africa (supports local MTN/Airtel routes seamlessly). |
| **Templating Engine** | **Handlebars** (`hbs`) | Used internally by the mailer to render beautiful, branded HTML emails containing the dynamic activation link. |
| **Token Generation** | `@nestjs/jwt` | Reusing the existing library to generate exactly 24-hour lifespan Activation Tokens. |
| **Real-time Engine** | `@nestjs/websockets` / Socket.io | To push real-time `status: Active` updates down to the frontend Admin Dashboard instantly as users onboard. |
| **Async Queueing** | `@nestjs/bull` (Redis) | *Recommended:* Pushing email/SMS jobs into a background Redis queue ensures the Admin UI doesn't freeze or timeout while waiting for external APIs to respond. |

## 5. Acceptance Criteria (Developer Checklist)
- [ ] New users default to `is_active: false` upon creation.
- [ ] An email or SMS is successfully, asynchronously dispatched to the new user without delaying the Admin's UI response.
- [ ] The Activation Link validates correctly and expires gracefully (e.g., after 24 hours), returning a clear UI message.
- [ ] Successful activation flips the user's database status to `is_active: true`.
- [ ] Attempting to login through the standard route *before* activation returns a `403 Forbidden - Account Inactive`.

## 6. PM Open Questions / Clarifications
- *All initial PM clarifications regarding Token Lifespan (24h), Delivery Priorities (Toggle UI), and Security Action (Set Password) have been confirmed and integrated into this technical task.*
