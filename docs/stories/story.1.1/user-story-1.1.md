# User Story 1.1: Role-Based Access Control (RBAC) Setup

## 1. Meta Information
- **Epic:** Epic 1: Identity & Access Management (IAM)
- **Story Priority:** High (Blocker for all other Epics)
- **Story Point Estimation:** Unestimated (To be sized by Engineering)
- **Status:** Draft

## 2. User Story
**As an** Admin,
**I want to** create user accounts and assign specific operational roles (Manager, Picker, Driver, Finance),
**So that** users only have access to the data and actions relevant to their job function, ensuring platform security and FDA compliance.

---

## 3. Business Context
In GreenFlow's operational model, an incorrect action (e.g., a Driver altering inventory counts or a Picker modifying supplier prices) can lead to systemic financial loss and regulatory failure (Rwanda FDA). Establishing a strict Role-Based Access Control (RBAC) perimeter is the foundation for our "Digital Twin" warehouse.

---

## 4. Acceptance Criteria (BDD Format)

### Scenario 1: Creating a new user with a specific role
**Given** the Admin is logged into the IAM dashboard
**When** the Admin submits a new user form with the role "Warehouse Picker"
**Then** the system should create the account with a default `is_active: false` status
**And** automatically dispatch an activation Email or SMS to the new user containing a secure login link.

### Scenario 1.5: Activating a newly created user account
**Given** a newly provisioned user clicks the activation link in their Email/SMS
**When** they successfully authenticate or set their initial password
**Then** the system should update their account status to `is_active: true`
**And** immediately restrict their API access exclusively to their assigned role's endpoints (e.g., Inventory).

### Scenario 2: Preventing unauthorized access (API Level)
**Given** a user with the role "Driver" is authenticated via JWT
**When** the Driver attempts to send a `PATCH` request to update a Supplier's Price Book
**Then** the API must return a `403 Forbidden` response
**And** the unauthorized attempt must be logged in the `Audit_Logs` table.

### Scenario 3: Preventing Privilege Escalation
**Given** an Admin is assigning roles to a new user
**When** the Admin attempts to create another "Super Admin"
**Then** the system must require secondary confirmation 
**And** optionally trigger an alert to the platform's root stakeholders.

---

## 5. Technical Tasks (Developer Implementation)
- [ ] **Task 1:** [Set up PostgreSQL `Users` table and `Roles` ENUM/Table.](./task-1.1.1.md)
- [ ] **Task 2:** [Implement JWT-based authentication service (login/token generation) on the backend.](./task-1.1.2.md) The API must return a structured JSON response containing the `access_token` and the user's `role`.
- [ ] **Task 3:** [Develop RBAC Middleware for role-checking on Backend API endpoints.](./task-1.1.3.md)
- [ ] **Task 4:** [Integrate Auth.js on the frontend using a `CredentialsProvider` to forward login requests to the backend.](./task-1.1.4.md) Configure Auth.js session callbacks to securely manage the JWT and user role in the frontend session for role-based routing guards.
- [ ] **Task 5:** [Create the frontend Admin UI for User Management (Create, Read, Update, Deactivate users).](./task-1.1.5.md)
- [ ] **Task 6:** [Implement Automated Credential Dispatch (Email/SMS) on the NestJS backend.](./task-1.1.6.md)

---

## 6. Edge Cases & Risk Mitigation
- **What if a user's role needs to change mid-shift?** 
  - *Mitigation:* The system must invalidate the active JWT upon a role change, forcing the user to re-authenticate to receive their new permissions.
- **What if an Admin accidentally deletes a user?**
  - *Mitigation:* **Hard deletes are strictly forbidden** per our database constraints. The system must only perform a "soft delete" (e.g., setting `is_active = false`), preserving the user's historical audit trail.

---

## 7. Dependencies
- Dependent on finalizing the `database-schema.md` for the Users and Audit Logs tables.
- Blocks execution of Epic 3 (Quality Control) and Epic 4 (Inventory), as those modules require authenticated users.
