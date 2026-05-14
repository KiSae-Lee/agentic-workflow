# tech-debt.md Template

Template + schema for `spec/tech-debt.md`. This file is the **single source of truth** for technical debt in the project. It serves as the issue tracker in this spec-as-files workflow.

**Lifecycle:**
- **Sole writer:** the `subagent-driven-development` dispatcher (never the implementer subagents — they only report entries via their task reports). This avoids `git merge` conflicts across parallel worktrees.
- **Created** lazily by the dispatcher at the end of the first phase that produces any tech debt.
- **Updated** by the dispatcher at the end of every implementation phase: append Opened entries, move Closed entries, recompute Dashboard.
- **Read** by `planning` (15% rule allocation) and by `code-review` (DoD enforcement: TODO format, entry-exists check, deliberate-debt section).

**Never delete entries.** Closed debt moves from "Active" to "Closed" with the closing phase ID and commit reference. The file tells the full debt-history of the project.

---

## ID Convention

Each entry gets a monotonically increasing ID: `TD-001`, `TD-002`, ... The next ID is `max(existing) + 1`, scanning both Active and Closed sections. **IDs are never reused** — when an entry closes, its ID stays with it.

Code annotations use the matching ID:

```
// TODO(#TD-042): swap polling for webhook once provider supports it
// FIXME(#TD-017): off-by-one on empty-list edge case
// HACK(#TD-029): hardcoded retry count, lift to config in payment-v2
```

**Every new `TODO`/`FIXME`/`HACK` in production code MUST reference an existing `TD-NNN` ID.** No exceptions. The code-review skill enforces this.

---

## Schema (per entry)

Each Active entry uses this exact structure. Every field is required.

```markdown
### TD-NNN — [one-line summary]

| Field | Value |
|---|---|
| **Category** | `code` / `design` / `doc` / `infra` |
| **Fowler quadrant** | `reckless-deliberate` / `reckless-inadvertent` / `prudent-deliberate` / `prudent-inadvertent` |
| **Principal (cost)** | N MD |
| **Interest rate** | N% (rework/workaround effort per phase, observed or estimated) |
| **Blast radius** | High / Medium / Low |
| **Priority score** | (Interest × RiskWeight) ÷ Principal = N.N |
| **Priority label** | P0 / P1 / P2 / P3 |
| **Location** | `path/to/file.ext:LINE` or `<system/component name>` |
| **Owner** | self (or named contributor) |
| **Repayment target** | YYYY-MM-DD, milestone, or `unscheduled` |
| **Registered in** | phase folder name (e.g. `2026-05-12T09-30-00_authentication`) |

**Reason:** [why this debt was taken, especially for `*-deliberate` quadrants]

**Remediation sketch:** [1-3 sentences on what closing this looks like]
```

### Priority score formula (APPENDIX §9)

```
score = (interest_rate_percent × risk_weight) ÷ principal_md
risk_weight: High=3, Medium=2, Low=1
```

| Score | Label | Handling |
|---|---|---|
| ≥ 50 | `P0` | Address immediately or block ongoing work |
| 20 – 49 | `P1` | Include in this phase (15% rule) |
| 5 – 19 | `P2` | Include in next phase |
| < 5 | `P3` | Backlog — revisit at workflow-complete review |

### Fowler quadrant guidance

| Quadrant | Meaning | Default handling |
|---|---|---|
| `reckless-deliberate` | "Ship now, fix later" — known shortcut, no plan to repay | Highest repayment priority; requires deliberate-debt section in implementation-report.md |
| `reckless-inadvertent` | Discovered post-hoc that approach was wrong from the start | Schedule repayment + capture lesson in spec docs |
| `prudent-deliberate` | "Repay after milestone X" — planned, scheduled trade-off | Repay per schedule (15% rule applies) |
| `prudent-inadvertent` | Discovered a better approach later, original was reasonable | Refactor opportunistically (Boy Scout rule) |

---

## File Structure

```markdown
# Technical Debt — [Project Name]

> Single source of truth for technical debt. Schema and lifecycle defined in
> `.claude/skills/subagent-driven-development/tech-debt-template.md`.

## Dashboard

**Last updated:** YYYY-MM-DDTHH-MM-SS (phase: `<phase-folder>`)
**Open count:** N (P0: N · P1: N · P2: N · P3: N)
**Inflow (last phase):** N opened
**Outflow (last phase):** N closed
**Inflow / Outflow ratio:** N.N (target ≤ 1.0; sustained > 1.0 across phases → trigger workflow-complete review)

### Top 5 by priority score

| ID | Summary | Score | Label | Location |
|---|---|---|---|---|
| TD-NNN | ... | N.N | Pn | path:line |
| ... | | | | |

---

## Active

[Entries in priority-score descending order. Full schema above. If empty: `_No active debt._`]

---

## Closed

[Entries with status fields appended: closed in phase X, commit hash, brief outcome. If empty: `_No closed debt yet._`]
```

### Closed-entry format

When an entry moves from Active → Closed, **prepend** these fields and keep the rest of the schema intact:

```markdown
### TD-NNN — [original summary] [CLOSED]

| Field | Value |
|---|---|
| **Closed in phase** | `<phase-folder-name>` |
| **Closed at** | YYYY-MM-DDTHH-MM-SS |
| **Commit / PR** | `<commit hash>` (or PR ref if used) |
| **Outcome** | [1-2 sentences: what was done, what changed] |

[... original Active fields preserved below ...]
```

---

## Few-Shot Example

```markdown
# Technical Debt — Workspace Platform

> Single source of truth for technical debt. Schema and lifecycle defined in
> `.claude/skills/subagent-driven-development/tech-debt-template.md`.

## Dashboard

**Last updated:** 2026-05-20T17-30-00 (phase: `2026-05-20T14-00-00_workspace-invitation`)
**Open count:** 4 (P0: 0 · P1: 1 · P2: 2 · P3: 1)
**Inflow (last phase):** 3 opened
**Outflow (last phase):** 1 closed
**Inflow / Outflow ratio:** 3.0 (above target — surface in workflow-complete review)

### Top 5 by priority score

| ID | Summary | Score | Label | Location |
|---|---|---|---|---|
| TD-003 | payment gateway timeout → retry-with-backoff | 24.0 | P1 | `src/payment/gateway.go:142` |
| TD-001 | RBAC checks duplicated in controller + service | 6.0 | P2 | `src/middleware/rbac.ts`, `src/auth/auth-service.ts` |
| TD-002 | Hardcoded 7-day invitation expiry | 6.0 | P2 | `src/invitation/invitation-service.ts:58` |
| TD-004 | Prisma client gen adds 3s to cold start | 1.5 | P3 | `prisma/schema.prisma` |

---

## Active

### TD-003 — payment gateway timeout → retry-with-backoff

| Field | Value |
|---|---|
| **Category** | `code` |
| **Fowler quadrant** | `prudent-deliberate` |
| **Principal (cost)** | 1 MD |
| **Interest rate** | 8% (timeout-related rework averaged 0.4 MD per phase across last 3 phases) |
| **Blast radius** | High |
| **Priority score** | (8 × 3) ÷ 1 = 24.0 |
| **Priority label** | P1 |
| **Location** | `src/payment/gateway.go:142` |
| **Owner** | self |
| **Repayment target** | milestone `payment-v2` |
| **Registered in** | `2026-05-12T09-30-00_authentication` |

**Reason:** Tight deadline on auth phase; current naive timeout works for happy path but causes 0.3% transaction failures on transient provider issues.

**Remediation sketch:** Wrap gateway calls in an exponential-backoff retry with jitter (3 attempts, 100ms base). Add metric `payment.retry_count` to confirm steady-state retry rate stays below 2%.

---

### TD-001 — RBAC checks duplicated in controller and service layers

| Field | Value |
|---|---|
| **Category** | `design` |
| **Fowler quadrant** | `prudent-inadvertent` |
| **Principal (cost)** | 2 MD |
| **Interest rate** | 6% (every new endpoint requires duplicated role check) |
| **Blast radius** | Medium |
| **Priority score** | (6 × 2) ÷ 2 = 6.0 |
| **Priority label** | P2 |
| **Location** | `src/middleware/rbac.ts`, `src/auth/auth-service.ts` |
| **Owner** | self |
| **Repayment target** | unscheduled |
| **Registered in** | `2026-05-12T09-30-00_authentication` |

**Reason:** Initial design split auth and authorization across two layers without a single enforcement point. Discovered after second endpoint required identical role logic.

**Remediation sketch:** Consolidate role enforcement into the middleware layer. Remove role checks from service methods; update tests to assert via HTTP layer only.

---

## Closed

### TD-005 — Workspace name uniqueness case-sensitive [CLOSED]

| Field | Value |
|---|---|
| **Closed in phase** | `2026-05-20T14-00-00_workspace-invitation` |
| **Closed at** | 2026-05-20T16-15-00 |
| **Commit / PR** | `a1b2c3d4` |
| **Outcome** | Normalized workspace names to lowercase at the DB-constraint level. Added migration `003_workspace_name_lower.sql`. |
| **Category** | `code` |
| **Fowler quadrant** | `prudent-inadvertent` |
| **Principal (cost)** | 1 MD |
| **Interest rate** | 4% |
| **Blast radius** | Medium |
| **Priority score** | (4 × 2) ÷ 1 = 8.0 |
| **Priority label** | P2 |
| **Location** | `src/workspace/workspace-service.ts:34` |
| **Owner** | self |
| **Repayment target** | `2026-05-20T14-00-00_workspace-invitation` |
| **Registered in** | `2026-05-12T09-30-00_authentication` |

**Reason:** Used Prisma default `String` field without explicit collation; only noticed after QA created `Engineering` and `engineering` as separate workspaces.

**Remediation sketch:** _(closed — see Outcome above)_
```

---

## Update Rules

When the `subagent-driven-development` **dispatcher** finishes a phase (after all worktrees have merged):

1. **Collect implementer reports.** Each implementer's task report contains an `Opened` list (full schema text per entry) and a `Closed` list (TD-NNN, commit hash, Outcome). Implementers do NOT modify `spec/tech-debt.md` directly.
2. **Append new entries** to `## Active` — one block per `TD-NNN` from any implementer's `Opened` list. Use the implementer's reported entry text verbatim; correct only obviously broken formatting.
3. **Move closed entries** from `## Active` to `## Closed`, prepending the closed-fields block (Closed in phase, Closed at, Commit / PR, Outcome — fill from the implementer's `Closed` report).
4. **Recompute the Dashboard:**
   - Open count and per-label counts from Active section.
   - Inflow = entries with `Registered in: <this phase>` (count of new TD-NNN added this phase).
   - Outflow = entries with `Closed in phase: <this phase>` (count of moved entries).
   - Ratio = Inflow ÷ Outflow (treat Outflow=0 as `∞` and flag).
   - Top 5 by priority score (descending), pulling from Active only.
5. **Update timestamp** with current phase folder name even if no Open/Close occurred this phase — the dashboard "Last updated" should always advance.

When `planning` reads `spec/tech-debt.md`:

- Sort Active entries by priority score descending.
- All `P0` entries → include in this phase's task.md, full stop.
- `P1` entries → include in priority order until estimated MD reaches ~15% of phase task estimate.
- Note in `plan.md` Phase scope section: "Tech-debt repayment: TD-xxx, TD-yyy (N MD ≈ N% of phase capacity)".

When `code-review` reviews implementation:

- Verify every new `// TODO(#TD-NNN): ...`, `// FIXME(#TD-NNN): ...`, `// HACK(#TD-NNN): ...` in the diff references an entry that exists in `spec/tech-debt.md`. Missing entry → Critical issue.
- Verify every `TODO`/`FIXME`/`HACK` without a `(#TD-NNN)` annotation is flagged. Bare TODOs → Critical issue.
- If implementation-report.md declares a `*-deliberate` debt was taken, verify the matching TD entry is registered with all schema fields populated. Missing fields → Important issue.
