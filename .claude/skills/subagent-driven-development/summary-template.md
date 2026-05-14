# summary.md Template

Template with few-shot example for `spec/summary.md`. This file is created after
the first phase's implementation completes and updated after each subsequent phase.
It is a **living document** — append new phase entries, never overwrite history.

## Structure

```markdown
# [Project Name] — Summary

## Current Status

**Phase:** [YYYY-MM-DDTHH-MM-SS_keyword or "Complete"]
**Last updated:** [date]
**ATDD progress:** [checked] / [total] acceptance criteria ([percentage]%)

## ATDD Progress by Feature

| Feature | Accepted | Total | Status |
|---|---|---|---|
| [Feature name] | [n] | [m] | [In Progress / Done / Not Started] |
| [Feature name] | [n] | [m] | [In Progress / Done / Not Started] |

## Phase History

### YYYY-MM-DDTHH-MM-SS_keyword — [Phase title]
**Date:** [completion date]
**Scope:** [Features / user stories covered]
**Outcome:** [what was built — 2-3 sentences]
**Key files changed:** [list of significant files/modules created or modified]

### YYYY-MM-DDTHH-MM-SS_keyword — [Phase title]
...

## Risks

| Risk | Severity | Mitigation | Owner |
|---|---|---|---|
| [description] | High / Medium / Low | [what we're doing about it] | [who owns it] |

## Technical Debt

Single source of truth: **`spec/tech-debt.md`** (Dashboard + Active + Closed sections, full schema per `.claude/skills/subagent-driven-development/tech-debt-template.md`).

This section is a *summary pointer*, not a duplicate ledger. Cite open count + top items only.

**Open count:** N (P0: N · P1: N · P2: N · P3: N) — see `spec/tech-debt.md` for full detail.

**Inflow / Outflow trend (last 3 phases):**

| Phase | Opened | Closed | Ratio |
|---|---|---|---|
| [phase folder] | N | N | N.N |
| [phase folder] | N | N | N.N |
| [phase folder] | N | N | N.N |

**Top 3 open by priority score:**

| ID | Summary | Score | Label |
|---|---|---|---|
| TD-NNN | [one-line] | N.N | Pn |

## Open Issues

| Issue | Found in | Blocking? | Notes |
|---|---|---|---|
| [description] | [YYYY-MM-DDTHH-MM-SS_keyword] | Yes / No | [context] |
```

---

## Few-Shot Example

```markdown
# Workspace Platform — Summary

## Current Status

**Phase:** 2026-05-20T14-00-00_workspace-invitation
**Last updated:** 2026-05-20
**ATDD progress:** 8 / 23 acceptance criteria (35%)

## ATDD Progress by Feature

| Feature | Accepted | Total | Status |
|---|---|---|---|
| Authentication | 3 | 3 | Done |
| Workspace Management | 3 | 5 | In Progress |
| Member Invitation | 2 | 4 | In Progress |
| Billing Integration | 0 | 4 | Not Started |
| Customer Support Tools | 0 | 3 | Not Started |
| Notification System | 0 | 4 | Not Started |

## Phase History

### 2026-05-12T09-30-00_authentication — Authentication & Tenant Setup
**Date:** 2026-05-14
**Scope:** Feature: Authentication (all 3 user stories)
**Outcome:** Implemented Okta SSO integration with SAML 2.0, JWT token
issuance (15min expiry + refresh cookie), tenant auto-provisioning on first
login, and RBAC middleware with four roles (owner, admin, member, viewer).
**Key files changed:**
- `src/auth/okta-provider.ts` — SSO integration
- `src/auth/jwt-service.ts` — token issuance and validation
- `src/middleware/rbac.ts` — role-based access control
- `prisma/migrations/001_tenant_user.sql` — tenant and user tables
- `tests/auth/` — 12 unit tests, 4 integration tests

### 2026-05-20T14-00-00_workspace-invitation — Workspace & Invitation (in progress)
**Date:** ongoing
**Scope:** Feature: Workspace Management (3 of 5 stories), Feature: Member Invitation (2 of 4 stories)
**Outcome:** Workspace CRUD implemented with tenant-scoped isolation.
Invitation flow partially complete — send and accept work, revoke and
expiry not yet implemented.
**Key files changed:**
- `src/workspace/workspace-service.ts` — CRUD operations
- `src/invitation/invitation-service.ts` — send + accept flow
- `src/events/publishers.ts` — workspace.created, member.joined events
- `prisma/migrations/002_workspace_invitation.sql` — workspace, membership, invitation tables

## Risks

| Risk | Severity | Mitigation | Owner |
|---|---|---|---|
| Event bus reliability — SNS/SQS has no DLQ configured yet | High | Add DLQ in phase 03 before billing integration | Backend team |
| Okta rate limits — no throttling on token refresh | Medium | Implement token cache, track in phase 03 | Backend team |
| No load testing done yet — NFR targets unverified | Medium | Schedule k6 load test after phase 03 | QA |

## Technical Debt

Single source of truth: **`spec/tech-debt.md`**.

**Open count:** 4 (P0: 0 · P1: 1 · P2: 2 · P3: 1)

**Inflow / Outflow trend (last 3 phases):**

| Phase | Opened | Closed | Ratio |
|---|---|---|---|
| 2026-05-20T14-00-00_workspace-invitation | 3 | 1 | 3.0 |
| 2026-05-12T09-30-00_authentication | 2 | 0 | ∞ |

**Top 3 open by priority score:**

| ID | Summary | Score | Label |
|---|---|---|---|
| TD-003 | payment gateway timeout → retry-with-backoff | 24.0 | P1 |
| TD-001 | RBAC checks duplicated in controller + service | 12.0 | P2 |
| TD-002 | Hardcoded 7-day invitation expiry | 6.0 | P2 |

## Open Issues

| Issue | Found in | Blocking? | Notes |
|---|---|---|---|
| Invitation email lands in spam for Gmail users | 2026-05-20T14-00-00_workspace-invitation | No | Need to configure SPF/DKIM for SendGrid domain |
| Workspace name uniqueness is case-sensitive | 2026-05-20T14-00-00_workspace-invitation | No | "Engineering" and "engineering" are treated as different — should normalize |
| Prisma client generation adds 3s to cold start | 2026-05-12T09-30-00_authentication | No | Acceptable for now, may need prisma generate optimization for Lambda if we move to serverless |
```

---

## Update Rules

When updating summary.md after a phase completes:

1. **Update "Current Status"** — bump phase number, date, ATDD progress counts
2. **Update "ATDD Progress by Feature"** — recalculate accepted counts, update statuses
3. **Add new Phase History entry** — append, never modify previous entries
4. **Review Risks** — remove resolved risks, add newly discovered ones, update severity if changed
5. **Review Technical Debt** — add new items from this phase, note if previous items were remediated (move to Phase History, don't delete)
6. **Review Open Issues** — add new, resolve old (move resolution to Phase History)

**Never delete history.** If a risk was resolved or debt was paid, note it in the Phase History entry where it was addressed. The summary must tell the full story of the project's evolution.
