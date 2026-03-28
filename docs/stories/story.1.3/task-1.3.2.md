# Task 1.3.2: Build Password Reset Frontend Flow

## 1. Meta Information
- **Parent Story:** [User Story 1.3: Password Reset & Session Management](./user-story-1.3.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Jira:** FL-27
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 4h

## 2. Objective
Build the Next.js frontend pages for the complete password reset journey — from the initial "Forgot Password?" link on the login page, through the email submission screen, to the final new-password confirmation form loaded from the reset link.

## 3. Implementation Requirements

### 3.1. Page 1: Reset Request (`/auth/forgot-password`)
- A simple centered form with a single email input field.
- On submit: call `POST /auth/reset-password/request`, disable the button and show a loading state.
- On any response (success or error): show a generic success message (`"If this email is registered, you will receive a link shortly."`) — never indicate whether the email was found.
- Add a "Back to Login" link.

### 3.2. Page 2: New Password Form (`/auth/reset-password?token={token}`)
- This page is loaded from the reset link in the user's email.
- On mount: validate the `token` query parameter exists (redirect to `/auth/forgot-password` with an error toast if missing).
- Display two password inputs: "New Password" and "Confirm Password".
- Client-side validation: passwords must match, minimum 8 characters, at least 1 uppercase and 1 number.
- On submit: call `POST /auth/reset-password/confirm` with `{ token, new_password }`.
- On `200`: show success toast, redirect to `/auth/login` after 2 seconds.
- On `400` (expired/invalid token): show error toast "This reset link is invalid or expired. Request a new one." with a CTA back to `/auth/forgot-password`.

### 3.3. Login Page Integration
- Add a "Forgot password?" link below the login form that routes to `/auth/forgot-password`.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Submitting any email on the forgot-password page shows the generic success message.
- [ ] Loading the reset page with an invalid token redirects to forgot-password with an error message.
- [ ] Submitting mismatched passwords shows a client-side validation error before calling the API.
- [ ] Successful password reset redirects to `/auth/login` and clears the Auth.js session.
- [ ] The reset page is not accessible to authenticated users (redirect to dashboard if session exists).

## 5. Dependencies
- Depends on Task 1.3.1 (Backend reset endpoints).
- Depends on Task 1.1.4 (Auth.js frontend integration) for session clearing on successful reset.
