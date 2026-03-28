# Technical Specification

**Project Name:** {{PROJECT_NAME}}
**Version:** {{VERSION}}
**Date:** {{DATE}}
**Author:** {{AUTHOR}}
**Status:** {{STATUS}}

---

## Overview

### Problem Statement
{{PROBLEM_STATEMENT}}

### Proposed Solution
{{SOLUTION_OVERVIEW}}

### Goals
- {{GOAL_1}}
- {{GOAL_2}}
- {{GOAL_3}}

---

## Scope

### In Scope
- {{IN_SCOPE_1}}
- {{IN_SCOPE_2}}
- {{IN_SCOPE_3}}

### Out of Scope
- {{OUT_OF_SCOPE_1}}
- {{OUT_OF_SCOPE_2}}
- {{OUT_OF_SCOPE_3}}

---

## Requirements

### Functional Requirements

#### FR-001: {{FR_1_TITLE}} [MUST/SHOULD/COULD]
{{FR_1_DESCRIPTION}}

**Acceptance Criteria:**
- {{FR_1_AC_1}}
- {{FR_1_AC_2}}

---

#### FR-002: {{FR_2_TITLE}} [MUST/SHOULD/COULD]
{{FR_2_DESCRIPTION}}

**Acceptance Criteria:**
- {{FR_2_AC_1}}
- {{FR_2_AC_2}}

---

#### FR-003: {{FR_3_TITLE}} [MUST/SHOULD/COULD]
{{FR_3_DESCRIPTION}}

**Acceptance Criteria:**
- {{FR_3_AC_1}}
- {{FR_3_AC_2}}

---

### Non-Functional Requirements

#### NFR-001: Performance [MUST/SHOULD]
{{NFR_PERF_DESCRIPTION}}

**Target:** {{NFR_PERF_TARGET}}

---

#### NFR-002: Security [MUST/SHOULD]
{{NFR_SEC_DESCRIPTION}}

**Requirements:**
- {{NFR_SEC_REQ_1}}
- {{NFR_SEC_REQ_2}}

---

#### NFR-003: Scalability [MUST/SHOULD]
{{NFR_SCALE_DESCRIPTION}}

**Target Load:** {{NFR_SCALE_TARGET}}

---

## Technical Approach

### Architecture Overview

> **AgriFlow Rwanda** — Phase 1 uses a **Service-Modular Monolith** architecture.
> Replace this note with spec-specific architecture details.

**Pattern:** NestJS modules co-deployed as a single service (monolith), but with strict module boundaries to enable extraction to microservices in Phase 2+.

```
┌─────────────────────────────────────────────────────────┐
│                   NestJS Application                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │  IAM     │  │Inventory │  │ Orders   │  │Logistics│ │
│  │ Module   │  │ Module   │  │ Module   │  │ Module  │ │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘ │
│                     PostgreSQL (RDS)                    │
└─────────────────────────────────────────────────────────┘
        ↑ REST API            ↑ REST API
┌───────────────┐     ┌────────────────────┐
│  Next.js Web  │     │  Flutter Mobile App│
│  (Admin/PM)   │     │  (Picker / Driver) │
└───────────────┘     └────────────────────┘
```

### Key Technologies

> **AgriFlow Rwanda** tech stack — pre-filled. Add spec-specific libraries as needed.

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Backend** | NestJS (Node.js) | REST API, business logic, RBAC middleware |
| **Database** | PostgreSQL (AWS RDS) | Relational integrity for ledger + inventory |
| **Frontend** | Next.js + Tailwind + React Query + Zustand | Admin portal, order management, reporting |
| **Mobile** | Flutter (or React Native) | Offline-first picker + driver apps |
| **Infrastructure** | AWS (RDS, S3, Lambda), Docker | Cloud hosting, file storage, containerization |
| **Auth** | JWT + NestJS Guards | RBAC enforcement at API boundary |
| **File Storage** | AWS S3 | QC rejection photos, PoD images |
| **Sync** | Local SQLite + REST sync | Offline-first mobile data persistence |

### Components

#### Component 1: {{COMPONENT_1_NAME}}
**Purpose:** {{COMPONENT_1_PURPOSE}}

**Responsibilities:**
- {{COMPONENT_1_RESP_1}}
- {{COMPONENT_1_RESP_2}}

**Interfaces:**
- {{COMPONENT_1_INTERFACE_1}}
- {{COMPONENT_1_INTERFACE_2}}

---

#### Component 2: {{COMPONENT_2_NAME}}
**Purpose:** {{COMPONENT_2_PURPOSE}}

**Responsibilities:**
- {{COMPONENT_2_RESP_1}}
- {{COMPONENT_2_RESP_2}}

**Interfaces:**
- {{COMPONENT_2_INTERFACE_1}}
- {{COMPONENT_2_INTERFACE_2}}

---

### Data Model

#### Entity 1: {{ENTITY_1_NAME}}
```
{{ENTITY_1_SCHEMA}}
```

#### Entity 2: {{ENTITY_2_NAME}}
```
{{ENTITY_2_SCHEMA}}
```

### API Design

#### Endpoint 1: {{ENDPOINT_1}}
**Method:** {{METHOD_1}}
**Purpose:** {{PURPOSE_1}}

**Request:**
```json
{{REQUEST_1_EXAMPLE}}
```

**Response:**
```json
{{RESPONSE_1_EXAMPLE}}
```

---

#### Endpoint 2: {{ENDPOINT_2}}
**Method:** {{METHOD_2}}
**Purpose:** {{PURPOSE_2}}

**Request:**
```json
{{REQUEST_2_EXAMPLE}}
```

**Response:**
```json
{{RESPONSE_2_EXAMPLE}}
```

---

## Ledger Design

> Include this section for any spec touching inventory, stock movements, or financial transactions (Epics 4, 5, 6, 8).

Every stock movement in AgriFlow is a **double-entry ledger record** — each entry has both a source and destination location. This eliminates stock "appearing" or "disappearing" and ensures instant reconciliation.

### Movement Record Pattern

```sql
CREATE TABLE stock_movements (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  batch_id         UUID NOT NULL REFERENCES inventory_batches(id),
  movement_type    VARCHAR(50) NOT NULL,  -- RECEIVE, PICK, TRANSFER, ADJUST, WRITE_OFF
  source_location  VARCHAR(100),          -- NULL only for initial receive
  dest_location    VARCHAR(100),          -- NULL only for write-off
  quantity         DECIMAL(10,3) NOT NULL CHECK (quantity > 0),
  unit             VARCHAR(20) NOT NULL,
  performed_by     UUID NOT NULL REFERENCES users(id),
  performed_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
  reference_id     UUID,                  -- links to order_id, qc_inspection_id, etc.
  notes            TEXT,
  is_active        BOOLEAN NOT NULL DEFAULT true
);
-- Constraint: at least one of source or destination must be set
ALTER TABLE stock_movements
  ADD CONSTRAINT chk_ledger_entry CHECK (
    source_location IS NOT NULL OR dest_location IS NOT NULL
  );
```

**Rule:** No inventory mutation without a corresponding `stock_movements` row. APIs that update batch quantities must insert to this table in the same transaction.

---

## Offline Behavior

> Include this section for any spec touching the mobile apps (Epics 4, 7 — warehouse picker and driver).

AgriFlow mobile apps must operate without network connectivity for a minimum of **8 continuous hours** (PRD constraint C4).

### Offline Strategy

| Concern | Strategy |
|---------|----------|
| **Data storage** | SQLite on-device (Flutter) or IndexedDB (React Native) for pending operations |
| **Sync trigger** | On network reconnect: upload pending queue, then pull server delta |
| **Conflict resolution** | Server-wins for inventory counts; last-write-wins for status fields with timestamp comparison |
| **Queue durability** | Pending operations persisted to local DB — survive app restart |
| **User feedback** | Offline indicator in UI; pending operation count displayed |

### Sync Queue Record Pattern

```typescript
interface PendingOperation {
  id: string;              // local UUID
  type: 'QC_SUBMIT' | 'POD_CAPTURE' | 'BATCH_RECEIVE' | 'PICK_CONFIRM';
  payload: object;         // full API request body
  createdAt: string;       // ISO timestamp
  retryCount: number;      // max 3 before flagging for manual review
  status: 'pending' | 'syncing' | 'failed';
}
```

**Acceptance requirement:** Any story touching mobile must include a test scenario: "Given the device is offline, when [action is performed], then the operation is queued locally and synced when connectivity is restored."

---

## Implementation Considerations

### Design Patterns
- {{PATTERN_1}}: {{PATTERN_1_RATIONALE}}
- {{PATTERN_2}}: {{PATTERN_2_RATIONALE}}

### Error Handling
{{ERROR_HANDLING_APPROACH}}

### Logging and Monitoring
{{LOGGING_APPROACH}}

**Key Metrics to Track:**
- {{METRIC_1}}
- {{METRIC_2}}
- {{METRIC_3}}

### Configuration Management
{{CONFIG_APPROACH}}

---

## Testing Strategy

### Unit Testing
**Coverage Target:** {{UNIT_TEST_COVERAGE}}%

**Focus Areas:**
- {{UNIT_TEST_AREA_1}}
- {{UNIT_TEST_AREA_2}}

### Integration Testing
**Scenarios:**
1. {{INTEGRATION_SCENARIO_1}}
2. {{INTEGRATION_SCENARIO_2}}
3. {{INTEGRATION_SCENARIO_3}}

### Performance Testing
**Load Profile:** {{LOAD_PROFILE}}

**Success Criteria:**
- {{PERF_CRITERION_1}}
- {{PERF_CRITERION_2}}

### Security Testing
**Tests Required:**
- {{SECURITY_TEST_1}}
- {{SECURITY_TEST_2}}
- {{SECURITY_TEST_3}}

---

## Deployment

### Deployment Strategy
{{DEPLOYMENT_STRATEGY}}

### Environment Requirements
- **Development:** {{DEV_REQUIREMENTS}}
- **Staging:** {{STAGING_REQUIREMENTS}}
- **Production:** {{PROD_REQUIREMENTS}}

### Rollout Plan
1. {{ROLLOUT_STEP_1}}
2. {{ROLLOUT_STEP_2}}
3. {{ROLLOUT_STEP_3}}

### Rollback Procedure
{{ROLLBACK_PROCEDURE}}

---

## Dependencies

### External Dependencies
| Dependency | Version | Purpose | Risk |
|------------|---------|---------|------|
| {{DEP_1}} | {{VERSION_1}} | {{PURPOSE_1}} | {{RISK_1}} |
| {{DEP_2}} | {{VERSION_2}} | {{PURPOSE_2}} | {{RISK_2}} |

### Internal Dependencies
- {{INTERNAL_DEP_1}}
- {{INTERNAL_DEP_2}}

---

## Assumptions and Constraints

### Assumptions
1. {{ASSUMPTION_1}}
2. {{ASSUMPTION_2}}
3. {{ASSUMPTION_3}}

### Constraints
1. {{CONSTRAINT_1}}
2. {{CONSTRAINT_2}}
3. {{CONSTRAINT_3}}

---

## Timeline

### Milestones
| Milestone | Target Date | Deliverables |
|-----------|-------------|--------------|
| {{MILESTONE_1}} | {{DATE_1}} | {{DELIVERABLE_1}} |
| {{MILESTONE_2}} | {{DATE_2}} | {{DELIVERABLE_2}} |
| {{MILESTONE_3}} | {{DATE_3}} | {{DELIVERABLE_3}} |

### Tasks Breakdown
1. **{{TASK_1}}** - {{TASK_1_ESTIMATE}}
2. **{{TASK_2}}** - {{TASK_2_ESTIMATE}}
3. **{{TASK_3}}** - {{TASK_3_ESTIMATE}}
4. **{{TASK_4}}** - {{TASK_4_ESTIMATE}}

**Total Estimated Effort:** {{TOTAL_ESTIMATE}}

---

## Risks and Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| {{RISK_1}} | {{IMPACT_1}} | {{PROB_1}} | {{MITIGATION_1}} |
| {{RISK_2}} | {{IMPACT_2}} | {{PROB_2}} | {{MITIGATION_2}} |
| {{RISK_3}} | {{IMPACT_3}} | {{PROB_3}} | {{MITIGATION_3}} |

---

## Success Criteria

- [ ] {{SUCCESS_CRITERION_1}}
- [ ] {{SUCCESS_CRITERION_2}}
- [ ] {{SUCCESS_CRITERION_3}}
- [ ] All functional requirements implemented
- [ ] All non-functional requirements met
- [ ] All tests passing
- [ ] Documentation complete
- [ ] Code reviewed and approved

---

## Appendix

### Glossary
| Term | Definition |
|------|------------|
| {{TERM_1}} | {{DEFINITION_1}} |
| {{TERM_2}} | {{DEFINITION_2}} |

### References
1. {{REFERENCE_1}}
2. {{REFERENCE_2}}
3. {{REFERENCE_3}}

### Diagrams
_[Attach architecture diagrams, flow charts, sequence diagrams]_

---

**Document End**
