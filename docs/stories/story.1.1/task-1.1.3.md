# Task 1.1.3: Develop RBAC Middleware for Backend API

## 1. Meta Information
- **Parent Story:** [User Story 1.1: Role-Based Access Control (RBAC) Setup](./user-story-1.1.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** [To be estimated by Engineering]

## 2. Objective
Implement reusable backend Middleware/Guards to enforce Role-Based Access Control (RBAC) across all API endpoints. This middleware will intercept incoming requests, validate the JWT, extract the user's role, and verify if the role is authorized to access the requested resource.

## 3. High-Level Requirements

### 3.1. JWT Verification
- The middleware must parse the `Authorization: Bearer <token>` header from incoming requests.
- Verify the structural integrity and signature of the JWT using the system's secret key.
- Verify that the token has not expired (`exp` claim).

### 3.2. Role Extraction & Authorization
- Extract the `role` and `sub` (user ID) from the valid token's payload.
- Compare the extracted `role` against an array of allowed roles defined at the endpoint level (e.g., a decorator or route configuration).
- If the token is invalid, missing, or expired, return `401 Unauthorized`.
- If the token is valid but the user's role is not authorized for the endpoint, return `403 Forbidden`.

### 3.3. Request Context Augmentation
- If the request is authorized, attach the user's information (at minimum `user_id` and `role`) to the request object (e.g., `req.user`). 
- This ensures downstream controllers and the future Audit Log Observer can easily access the identity of the user performing the action without re-parsing the token.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Middleware is created and can be globally or selectively applied to routes.
- [ ] `401 Unauthorized` is returned for requests lacking a token, or possessing an invalid/expired token.
- [ ] `403 Forbidden` is returned for authenticated users attempting to access endpoints outside their role permissions.
- [ ] The authenticated user's ID and role are successfully attached to the request context.
- [ ] Unit tests are written to mock incoming requests and verify the correct status codes are returned for various roles.

## 5. Security & Compliance Context
- **Accountability:** This task is a prerequisite for Epic 1.2 (Audit Logging). Without the middleware injecting the `user_id` into the request context, it is impossible to trace critical inventory mutations back to specific operators, failing Rwanda FDA compliance requirements.
- **Strict Perimeter:** Ensure there is a failsafe or default "closed" posture architecture, where new endpoints are protected by default and must be explicitly opened.

## 6. Open Questions / Clarifications
- *userAskQuestion*: Do we need more granular permissions (Resource-level access) beyond just Role-level access for Phase 1 MVP? For example, can *any* Manager view *all* supplier price books, or should they be restricted to specific regions/suppliers?
- *userAskQuestion*: Should failed RBAC authorization attempts (`403 Forbidden`) be logged into a security event stream or the main `Audit_Logs` table to identify potential malicious internal activity or privilege escalation attempts?
