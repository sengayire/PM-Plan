# User Story 2.2: Supplier & Product Mapping (Price Books)

## 1. Meta Information
- **Epic:** [Epic 2: Master Product & Supplier Catalog](../../epics/EPICS.md)
- **Story Priority:** High
- **Story Point Estimation:** Unestimated (To be sized by Engineering)
- **Status:** Draft

## 2. User Story
**As a** Buyer (Procurement),
**I want to** map established AgriFlow products to specific registered suppliers along with their agreed pricing models,
**So that** warehouse Receiving clerks know exactly who is approved to supply what, and our platform accurately tracks supplier-specific cost inputs for financial and commission reporting.

---

## 3. Business Context
A kilogram of tomatoes from 'Supplier A The Farmer' might cost 500 RWF, while 'Supplier B Aggregator' charges 550 RWF/kg. To manage sourcing efficiency and control operational spend, AgriFlow must tie the universal `Product_Master` SKU entities to individual registered Suppliers. This data mapping serves as the supplier's "System Price Book", preventing receiving teams from entering ad-hoc arbitrary prices and enabling Finance to accurately forecast operational capital outflow.

---

## 4. Acceptance Criteria (BDD Format)

### Scenario 1: Mapping a Product to a Supplier with an Agreed Price
**Given** the Buyer is viewing a specific Supplier's system profile
**When** the Buyer adds a new mapping for "Tomatoes" at an agreed price of "500" and currency "RWF"
**Then** the system should successfully save this relational mapping
**And** the mapping should reflect active immediately so that the Smart Receiving portal suggests "500 RWF" when this product arrives from this supplier.

### Scenario 2: Preventing Duplicate Mappings
**Given** a price mapping already exists between "Supplier A" and "Tomatoes"
**When** the Buyer attempts to create a completely new mapping for "Supplier A" and "Tomatoes"
**Then** the UI should throw a validation error
**And** prompt the user to *Update* the price of the existing mapping instead of creating a conflicting duplicate database row.

### Scenario 3: Viewing a Supplier's Entire Price Book
**Given** a supplier has multiple mapped product catalogs
**When** the Buyer navigates to the "Supplier Price Book" or "Catalogs" tab within their profile
**Then** the Buyer should clearly see a data table of all products supplied by that vendor, their active agreed prices, and the timestamp of when the price was last modified.

---

## 5. Technical Tasks (Developer Implementation)
- [ ] **Task 1:** Create `Suppliers` PostgreSQL schema. (Attributes: `id`, `name`, `contact_number`, `location`, `bank_details`, `is_active`).
- [ ] **Task 2:** Create the `Supplier_Product_Map` junction PostgreSQL table. (Attributes: `supplier_id`, `product_id`, `agreed_price` (decimal), `currency` (VARCHAR, e.g., 'RWF'), `updated_at`). Enforce a unique compound index on `(supplier_id, product_id)` to prevent duplicates.
- [ ] **Task 3:** Implement Backend API endpoints to manage Suppliers and their nested Product Mappings (`POST /suppliers`, `GET /suppliers/:id/products`, `PATCH /suppliers/mappings/:id`).
- [ ] **Task 4:** Build the Supplier Directory Frontend UI (List view, Search bar, and a 'Create Vendor' flow).
- [ ] **Task 5:** Build the "Supplier Price Book" Frontend UI (A nested view inside the Supplier Profile where a Buyer can search the global Product Catalog, select an item, and bind a price to it).

---

## 6. Edge Cases & Risk Mitigation
- **What if a supplier changes their price mid-season due to market shifts?**
  - *Mitigation:* Updating the `agreed_price` on the mapping table should ONLY affect future receiving batches. It must NEVER retroactively change the financial value of past inventory batches that have already been settled. 
- **What if a supplier is delisted or suspended?**
  - *Mitigation:* The supplier is marked `is_active: false`. Their existing price mappings should gracefully become dormant and un-selectable in future POs or Receiving Workflows, but their historical data remains intact.

---

## 7. Dependencies
- Dependent on User Story 2.1 (The global `Product_Master` catalog must exist and contain items before they can be assigned here).
- Unblocks Epic 3: Smart Receiving & Quality Control.

---

## 8. PM Open Questions / Clarifications
- **userAskQuestion:** Do we need to establish a formal "Contract / Initial Agreement Document" upload mechanism (PDF ingestion to S3) for the MVP when mapping these prices, or is the Buyer's application input legally trusted as the business source of truth?
- **userAskQuestion:** Will fresh produce market prices fluctuate frequently enough (e.g., daily) that we need to architect a robust *historical pricing log* (time-series pricing data) from day one, or is simply overwriting the single `agreed_price` field sufficient for MVP?
- **userAskQuestion:** Should there be a mandatory Maker/Checker approval workflow for amending an `agreed_price` (e.g., Buyer proposes price -> Finance Manager approves), or can authenticated Buyers change prices autonomously?
