---
name: story-validator
description: Validates generated AgriFlow story and task files against CLAUDE.md writing standards before commit. Checks BDD format, edge cases, RBAC notes, audit trail references, hour estimates, constraint compliance (C1–C5), and naming conventions. Run after story generation, before jira-sync and git commit.
allowed-tools: Read, Glob, Grep, Write, TodoWrite
argument-hint: "[story path, e.g. story.4.1 or story.4.* or docs/stories/]"
effort: low
context: fork
---

# Story Validator Skill — AgriFlow Rwanda

**Role:** Quality gate for all generated planning artifacts. Runs after `product-manager` story generation and before `jira-sync` + git commit.

---

## Boot Sequence

```
1. Read docs/context/agriflow-agent-context.md     ← validation rules reference
2. Identify target stories to validate (folder glob or explicit list)
3. Run all checks per story and task file
4. Write validation report to docs/context/validation-report.md
5. Report PASS or FAIL with specific issues
```

---

## Validation Workflow

### Input: Story folder(s) to validate

```
Validate a single story:     docs/stories/story.{E}.{S}/
Validate a sprint's stories: docs/stories/story.{E}.*/
Validate all stories:        docs/stories/
```

### Step 1 — Discover files

```
Glob: docs/stories/story.{E}.{S}/user-story-{E}.{S}.md   ← story file
Glob: docs/stories/story.{E}.{S}/task-{E}.{S}.*.md       ← task files
```

### Step 2 — Run story-level checks (see §A below)

### Step 3 — Run task-level checks per task file (see §B below)

### Step 4 — Write report and return result

---

## §A — Story-Level Checks

Run each check against `user-story-{E}.{S}.md`. Mark PASS or FAIL with line reference.

| # | Check | PASS condition | FAIL example |
|---|-------|---------------|--------------|
| S1 | **Story format** | Contains `AS A`, `I WANT TO`, `SO THAT` on the same line or consecutive lines | "User should be able to log in" |
| S2 | **Valid persona** | Persona matches one of: Admin, Procurement Manager, Warehouse QC Clerk, Warehouse Picker, Logistics Manager, Driver, Finance Manager, System Auditor, Supermarket Manager, Supplier | "As a user..." |
| S3 | **BDD acceptance criteria** | ≥3 criteria; each starts with `Given`, `When`, `Then` in that order | Bullet-point criteria without Given/When/Then |
| S4 | **Negative/error path** | At least 1 criterion covers a failure case (403, QUARANTINE, network offline, invalid input) | All criteria are happy-path only |
| S5 | **Edge cases section** | `## Edge Cases` section present with ≥2 items; each has a **Mitigation** | Only 1 edge case, or no mitigations |
| S6 | **Technical context section** | `## Technical Context` section present | Section absent |
| S7 | **Tables referenced** | Technical context names at least one table written or read | "data will be stored" with no table name |
| S8 | **RBAC constraint** | Technical context mentions a role AND what happens on unauthorized access (403 + Audit_Logs) | No role or access control mentioned |
| S9 | **Audit trail** | Technical context names which events are written to `Audit_Logs` | No audit trail reference for an inventory/financial/receiving story |
| S10 | **No hard deletes** | No reference to `DELETE`, permanent removal, or "delete the record" without `is_active: false` | "The record is deleted from the database" |
| S11 | **storage_type mention** | If story involves products or batches (Epics 3–9), `storage_type` is referenced somewhere | Epic 4 story with no storage_type mention |
| S12 | **Jira key in header** | Epic Jira key (FL-4 through FL-12) present in the header | No Jira key referenced |
| S13 | **Sprint assignment** | Sprint number present in header | Missing sprint |
| S14 | **Task list in story** | Story file contains a `## Tasks` section linking to task files | Tasks section absent |
| S15 | **Task count** | ≥3 task links in the Tasks section | Only 1 or 2 tasks |

**Epics where S9 (Audit trail) is REQUIRED:** All epics — every story in this system touches inventory, access, or financial data.

**Epics where S11 (storage_type) is REQUIRED:** Epics 3, 4, 5, 9.

---

## §B — Task-Level Checks

Run each check against every `task-{E}.{S}.{T}.md`. Mark PASS or FAIL.

| # | Check | PASS condition | FAIL example |
|---|-------|---------------|--------------|
| T1 | **Hour estimate present** | Contains `Xh` or `— Xh` or `**Estimate:** Xh` | No hour estimate anywhere in file |
| T2 | **Hour estimate reasonable** | Estimate is 1h–8h inclusive | "Estimate: 16h" — needs splitting |
| T3 | **Verb-first description** | Title and description start with an action verb (Create, Implement, Build, Write, Add, Configure, Define, Generate, Set up) | "JWT token handling" — no verb |
| T4 | **Story backlink** | Contains a relative link back to the parent `user-story-{E}.{S}.md` | No parent reference |
| T5 | **Role assigned** | `**Role:**` field present with one of: Backend, Frontend, Mobile, Infrastructure, Full-Stack | Role field missing |
| T6 | **Acceptance criteria** | At least 1 `- [ ]` checklist item under `## Acceptance` or `## Acceptance Criteria` | No acceptance checkboxes |
| T7 | **No hard delete instructions** | No `DELETE FROM`, `DROP TABLE`, or permanent removal without `is_active` | "Drop the old batch records" |
| T8 | **Specific, not vague** | Description contains a concrete noun (table name, API endpoint, component name, field name) | "Implement the backend logic" |

---

## §C — Structural Checks

| # | Check | PASS condition |
|---|-------|---------------|
| X1 | **File naming** | Story file: `user-story-{E}.{S}.md`; Task files: `task-{E}.{S}.{T}.md` |
| X2 | **Folder naming** | Folder: `story.{E}.{S}` (no other formats) |
| X3 | **No orphaned tasks** | Every task file in the folder is linked from the story's Tasks section |
| X4 | **No missing tasks** | Every task linked in the story's Tasks section has a corresponding file |

---

## Validation Report Format

Write to `docs/context/validation-report.md`:

```markdown
# Story Validation Report

**Date:** YYYY-MM-DD
**Validated by:** story-validator skill
**Scope:** [story.E.S | sprint-N | all]

## Summary

| Story | Story Checks | Task Checks | Result |
|-------|-------------|-------------|--------|
| story.{E}.{S} | X/15 passed | X/8 × N tasks passed | PASS / FAIL |

## Issues

### story.{E}.{S} — [PASS | FAIL]

**Story-level failures:**
- [S4] No negative/error path criterion — add a Given/When/Then for the 403 or QUARANTINE case
- [S11] No storage_type reference — this is an Epic 4 story; storage_type must be mentioned in Technical Context

**Task-level failures:**
- task-{E}.{S}.2: [T2] Estimate is 12h — split into two tasks ≤8h each
- task-{E}.{S}.3: [T3] Description "JWT handling" — rewrite to start with a verb

## Next Steps

- [ ] Fix flagged issues in story files
- [ ] Re-run story-validator
- [ ] On full PASS → run jira-sync skill
```

---

## Severity Levels

| Severity | Checks | Action |
|----------|--------|--------|
| **Blocker** — must fix before commit | S1, S2, S3, S5, S10, T1, T7, X1, X2 | Do not proceed to jira-sync or commit |
| **Required** — must fix before Jira sync | S4, S6, S7, S8, S9, S11, S12, S13, T2, T3, T4, T5 | Fix before syncing to Jira |
| **Advisory** — fix recommended | S14, S15, T6, T8, X3, X4 | Flag but do not block |

---

## Quick Reference — What Good Looks Like

Reference story for style: `docs/stories/story.1.1/user-story-1.1.md`
Reference task for style: `docs/stories/story.1.1/task-1.1.1.md`

Both of these pass all checks — use them as the gold standard when unsure.
