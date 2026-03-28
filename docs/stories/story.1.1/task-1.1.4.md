# Task 1.1.4: Frontend Auth.js Integration (RBAC)

## 1. Meta Information
- **Parent Story:** [User Story 1.1: Role-Based Access Control (RBAC) Setup](./user-story-1.1.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** [To be estimated by Engineering]

## 2. Objective
Integrate `Auth.js` (formerly NextAuth.js) into the Next.js frontend application. This integration will manage the user session natively on the client, securely store the JWT provided by the backend API, and make the user's operational role available throughout the frontend to conditionally render UI components and protect routes based on RBAC rules.

## 3. High-Level Requirements

### 3.1. Auth.js Configuration
- Initialize `Auth.js` within the Next.js application (App Router or Pages Router, depending on current architecture).
- Configure a `CredentialsProvider` to handle the login flow.
- The `authorize` function within the `CredentialsProvider` must:
  - Intercept the user's submitted email and password.
  - Forward these credentials via a `POST` request to the backend API (`/auth/login`) built in Task 1.1.2.
  - Parse the JSON response. If successful, extract the `access_token` and the `user` object (containing `id` and `role`).
  - Return this data to Auth.js to construct the session.

### 3.2. Session and JWT Callbacks
- Implement the `jwt` callback in the Auth.js configuration to persist the backend API's `access_token` and the user's `role` within the encrypted NextAuth token.
- Implement the `session` callback to expose the `role` and the `access_token` to the client-side session object. This allows frontend components (via `useSession` or `auth()`) to easily access the role for UI conditional rendering.

### 3.3. HTTP Client Interceptor (Optional but Recommended)
- Configure the frontend HTTP client (e.g., Axios or native fetch wrapper) to automatically attach the stored `access_token` as an `Authorization: Bearer <token>` header to all outgoing requests to the backend API.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `Auth.js` is successfully installed and initialized in the Next.js project.
- [ ] `CredentialsProvider` successfully authenticates against the custom backend API, not a direct database connection.
- [ ] The user's `role` and the backend `access_token` are successfully persisted in the Auth.js session token.
- [ ] The `useSession` hook (or equivalent server-side `auth()` function) correctly surfaces the user's role to frontend components.
- [ ] Attempting to login with incorrect credentials results in a handled error state on the frontend (no UI crash).

## 5. Security Context
- **Token Security:** Ensure the Auth.js session secret (`AUTH_SECRET`) is a strong, cryptographically secure random string stored securely in environmentally variables.
- **Offline Considerations:** While Auth.js handles standard web sessions well, clarify how this interacts with the offline-first requirements for the Driver/Picker mobile (or PWA) interfaces.

## 6. Open Questions / Clarifications
- *userAskQuestion*: For the mobile apps (Flutter/React Native), we won't be using Auth.js. Should we scope a separate companion task for the mobile authentication flow (Secure Storage + Dio/Axios interceptors), or is the mobile team handling their own ticketing?
- *userAskQuestion*: If a user's role is changed *while* they are logged in, Auth.js won't automatically know until the session is refreshed. Should we implement a forced session refresh mechanism natively, or rely on the backend API returning a `401/403` to trigger a client-side logout?
