To translate the high-level PRD into a **Jira-ready Backlog**, we break the project down into a standard hierarchy: **Epics** (Large bodies of work), **User Stories** (End-user value), and **Technical Tasks** (Developer implementation steps).

Below is the structured backlog for the **Phase 1: MVP (Year 1)**.

---

## 🏗 Project: AgriFlow Rwanda (MVP)

### Epic 1: Identity & Access Management (IAM)

**Goal:** Secure the platform and establish accountability via RBAC.

- **User Story 1.1:** As an Admin, I want to create user accounts with specific roles (Manager, Picker, Driver, Finance) so that users only access relevant data.
- **Technical Tasks:**
- Set up PostgreSQL `Users` and `Roles` tables.
- Implement JWT-based authentication service.
- Develop Middleware for role-checking on API endpoints.

- **User Story 1.2:** As a System Auditor, I want a non-editable log of all critical inventory changes so I can track discrepancies.
- **Technical Tasks:**
- Create an `Audit_Logs` table.
- Implement a global "Observer" to log POST/PATCH/DELETE requests on inventory entities.

---

### Epic 2: Master Product & Supplier Catalog

**Goal:** Standardize the data foundation for the entire supply chain.

- **User Story 2.1:** As a Procurement Manager, I want to define products with shelf-life and storage requirements so the system can calculate expiry automatically.
- **Technical Tasks:**
- Create `Product_Master` schema (attributes: `shelf_life_days`, `storage_temp_min`, `storage_temp_max`, `unit_of_measure`).
- Build CRUD UI for product management.

- **User Story 2.2:** As a Buyer, I want to map products to specific suppliers with agreed pricing.
- **Technical Tasks:**
- Create `Supplier_Product_Map` table with `agreed_price` and `currency`.

---

### Epic 3: Smart Receiving & Quality Control (RQC)

**Goal:** The "Front Gate" of the system to ensure quality and traceability.

- **User Story 3.1:** As a Warehouse Receiver, I want to log incoming shipments against a PO so I can verify what was delivered.
- **Technical Tasks:**
- Build Receiving Entry form.
- Implement logic to generate a unique `Batch_ID` using the logic: `[Date]-[SupplierID]-[SKU]`.

- **User Story 3.2:** As a QC Inspector, I want to upload photos and pass/fail a batch so that bad produce is blocked from inventory.
- **Technical Tasks:**
- Integrate AWS S3 or Google Cloud Storage for image hosting.
- Create `QC_Logs` table linked to `Batch_ID`.
- Implement logic: If `QC_Status` != 'PASSED', `Stock_Status` = 'QUARANTINE'.

---

### Epic 4: Ledger-Based Inventory Control

**Goal:** Real-time, high-integrity stock management.

- **User Story 4.1:** As an Inventory Manager, I want to see stock levels by batch and location so I can prevent spoilage.
- **Technical Tasks:**
- Implement **Double-Entry Ledger logic** (every stock move must have a `source_location` and `destination_location`).
- Build a background job to flag batches that are within 48 hours of expiry.

- **User Story 4.2:** As a Picker, I want the system to suggest the oldest batch first (FIFO) when I start an order.
- **Technical Tasks:**
- Develop a "Pick Suggestion" algorithm: `ORDER BY expiry_date ASC`.

---

### Epic 5: B2B Order & Settlement

**Goal:** Revenue generation and fulfillment.

- **User Story 5.1:** As a Supermarket Manager, I want to place orders via a simplified portal so I don't have to call the warehouse.
- **Technical Tasks:**
- Build a lightweight B2B Order UI.
- Implement real-time stock validation (prevent ordering more than what is "Available for Sale").

- **User Story 5.2:** As a Finance Officer, I want to generate invoices automatically upon delivery confirmation.
- **Technical Tasks:**
- Integrate a PDF generation library (e.g., Puppeteer or PDFKit).
- Trigger "Invoice Created" event upon "Proof of Delivery" upload.

---

### Epic 6: Last-Mile Delivery (Mobile Interface)

**Goal:** Logistics accountability.

- **User Story 6.1:** As a Driver, I want to see my daily delivery route and capture customer signatures on my phone.
- **Technical Tasks:**
- Develop a mobile-responsive "Driver View."
- Implement "Proof of Delivery" (PoD) module: Signature capture + GPS coordinate logging.

---

## 🛠 Strategic Technical Stack Recommendation

To ensure this scales as per your PRD, I recommend:

- **Backend:** Node.js (NestJS) or Python (FastAPI) for high-concurrency order processing.
- **Database:** PostgreSQL (Relational integrity is vital for ledgers).
- **Mobile:** Flutter or React Native (Cross-platform for Warehouse/Driver apps).
- **Infrastructure:** AWS (RDS for DB, S3 for QC photos, Lambda for automated expiry alerts).
