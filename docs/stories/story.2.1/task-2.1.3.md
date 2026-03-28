# Task 2.1.3: Product Catalog Data Table UI (Next.js)

## 1. Meta Information
- **Parent Story:** [User Story 2.1: Master Product Catalog Management](./user-story-2.1.md)
- **Epic:** Epic 2: Master Product & Supplier Catalog
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Construct the primary frontend interface where Procurement Managers visualize the entire Master Product Catalog. This Next.js React component must efficiently parse the paginated API data and provide intuitive controls for sorting and rapid searching of SKUs.

## 3. High-Level Requirements

### 3.1. Data Table Component
- Implement a robust data table matching the design system (e.g., utilizing an accessible headless UI or standard component library like Shadcn/MUI).
- **Columns to display:** `SKU Code`, `Name`, `Category`, `Shelf Life`, `Temperature Range`, `Unit`, and `Status` (`Active` / `Inactive`).
- Implement dynamic client-side or server-side pagination controls at the bottom of the table.

### 3.2. Filtering & Search Header
- Include a debounced text search input spanning the `Name` and `SKU Code` fields. (e.g., 500ms debounce before firing the API request to prevent spam).
- Include a Category dropdown filter.
- Include a Status toggle (e.g., "Show Inactive SKUs").

### 3.3. Visual Quality & API Integration
- Utilize standard hooks (`SWR` or `React Query`) to elegantly manage the fetching, caching, and loading states from `GET /products`.
- Visually distinguish deactivated products (e.g., by rendering row text in a faded gray and using a clear "Inactive" badge).
- Include an "Add New Product" primary call-to-action button that routes to the creation form.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] The API data flawlessly binds to the table rows.
- [ ] Entering text in the search bar triggers a filtered refresh of the table data.
- [ ] Pagination controls accurately navigate between pages of results.
- [ ] Loading states (skeletons or spinners) display cleanly while data is actively fetching.
- [ ] The route is guarded by Auth.js to prevent unauthenticated access.
