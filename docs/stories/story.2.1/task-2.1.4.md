# Task 2.1.4: Implement S3 Contract Document Upload for Supplier Agreements

## 1. Meta Information
- **Parent Story:** [User Story 2.1: Supplier Onboarding & Profile Management](./user-story-2.1.md)
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Jira:** FL-44
- **Estimate:** 6h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Implement S3-backed contract document storage for supplier agreements. The upload flow uses AWS pre-signed URLs to avoid routing large files through the NestJS application server, while keeping documents private and access-controlled.

## 3. Implementation Requirements

### 3.1. Upload Flow (Pre-signed URL Pattern)
1. Frontend calls `POST /suppliers/:id/contract/upload-url` with `{ fileName: string, contentType: string }`.
2. Backend generates a pre-signed PUT URL using `@aws-sdk/client-s3` `PutObjectCommand`.
3. S3 key format: `supplier-contracts/{supplierId}/{timestamp}-{sanitized-fileName}`.
4. Pre-signed URL expiry: **15 minutes** (PUT upload window).
5. Backend returns `{ uploadUrl, s3Key }` to the frontend.
6. Frontend PUTs the file directly to S3 using the pre-signed URL.
7. Frontend calls `POST /suppliers/:id/contract/confirm` with `{ s3Key }` to finalize.
8. Backend saves the S3 key to `supplier.contract_document_url` and writes an audit log entry.

### 3.2. Bucket & ACL Configuration
- Bucket: configured via `AWS_S3_BUCKET` env variable
- All objects uploaded under `supplier-contracts/` prefix must have **private ACL** — no public read.
- Enable server-side encryption: `AES256`.

### 3.3. Document Retrieval (Pre-signed GET URL)
- Endpoint: `GET /suppliers/:id/contract/view-url`
- Backend generates a pre-signed GET URL with **1-hour expiry** using `GetObjectCommand`.
- Returns `{ viewUrl }` — frontend opens in a new tab.
- Endpoint requires Procurement Manager or Admin role.

### 3.4. Validation
- Allowed content types: `application/pdf`, `image/jpeg`, `image/png`.
- Maximum file size: **10 MB** — enforced via `Content-Length` check on the `confirm` endpoint after upload.
- If content type or size is invalid, backend calls `DeleteObjectCommand` to clean up the orphaned S3 object and returns `422 Unprocessable Entity`.

### 3.5. Partial Failure Tolerance
- If the S3 upload fails or `confirm` is never called, the supplier profile is **not blocked** — `contract_document_url` remains null.
- The Documents tab on the Supplier Profile page (FL-45) displays a "No contract attached — Upload now" call-to-action.
- A clear error toast must inform the user if the upload failed.

### 3.6. Audit Logging
- Successful `confirm` call writes: `{ action: "CONTRACT_UPLOADED", entity: "Supplier", entity_id, changes: { s3Key } }`.
- Failed upload attempts are NOT written to audit log (no partial/failed audit entries).

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `POST /suppliers/:id/contract/upload-url` returns a valid S3 pre-signed PUT URL
- [ ] File uploaded via pre-signed URL lands in `supplier-contracts/{supplierId}/` with private ACL
- [ ] `POST /suppliers/:id/contract/confirm` updates `contract_document_url` and writes audit log
- [ ] `GET /suppliers/:id/contract/view-url` returns a 1-hour pre-signed GET URL
- [ ] Invalid file type returns `422`; orphaned S3 object is cleaned up
- [ ] Supplier profile saves successfully even if contract upload is skipped
- [ ] Error toast shown on upload failure in the UI

## 5. Dependencies
- **Requires:** FL-35 (Suppliers table — `contract_document_url` column)
- **Requires:** FL-43 (Supplier REST API — supplier record must exist)
- **Requires:** AWS S3 bucket provisioned with `supplier-contracts/` prefix policy
- **Unblocks:** FL-45 (Supplier Profile page — Documents tab)
