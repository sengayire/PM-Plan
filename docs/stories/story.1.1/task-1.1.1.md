# Task 1.1.1: Database Schema Setup - Users & Roles

## 1. Meta Information
- **Parent Story:** [User Story 1.1: Role-Based Access Control (RBAC) Setup](./user-story-1.1.md)
- **Epic:** Epic 1: Identity & Access Management (IAM)
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** [To be estimated by Engineering]

## 2. Objective
Create the foundational PostgreSQL database structure to support user authentication, role-based access control, and comprehensive audit logging for the AgriFlow platform. This ensures all system interactions can be traced back to a specific authorized user.

## 3. High-Level Requirements

### 3.1. Roles Definition
- Implement a structured mechanism (e.g., ENUM or a dedicated lookup table) to define the specific operational roles within AgriFlow:
  - `Admin`
  - `Manager`
  - `Warehouse Picker`
  - `Driver`
  - `QC Inspector`
  - `Finance`

### 3.2. Users Table Structure
- Create the core `users` table to store essential identifying and authentication data.
- **Expected Data Points:**
  - Unique Identifier (UUID preferred).
  - Credentials (e.g., securely hashed passwords).
  - Basic Profile Information (Name, Email, Phone Number).
  - Role association (linking the user to a defined role).
  - Status flag (e.g., `is_active` boolean) for account deactivation.
- **Critical System Constraint:** Hard deletions are strictly forbidden across the entire AgriFlow system. The schema design must support "soft deletion" (e.g., toggling the active status flag) to preserve the historical integrity of the ledger and audit trails.

### 3.3. Preparation for Audit Logging
- Ensure the user primary key is structured in a way that can be seamlessly referenced as a foreign key by the future `audit_logs` table. Every critical state change in the system will eventually need to be linked to a `user_id`.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Database migration scripts are created for the `roles` structure.
- [ ] Database migration scripts are created for the `users` table.
- [ ] Schema enforces basic data integrity constraints (e.g., unique email/phone, required fields).
- [ ] Migrations include an up/down (apply/rollback) mechanism and execute successfully on a clean local PostgreSQL instance.
- [ ] Initial seed data script is provided containing at least one default `Admin` user to allow for initial system login.

## 5. Security Context
- **Compliance Link:** This task directly supports Rwanda FDA compliance. If a user record could be fully deleted ("hard delete"), any past inventory movements or QC checks they conducted would become un-attributable, resulting in an immediate audit failure.
