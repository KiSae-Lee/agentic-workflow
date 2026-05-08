# Test Design Review Report

| Field | Value |
| --- | --- |
| **Target** | {ONE_LINE_TARGET_DESCRIPTION} |
| **Codebase path** | `{ABSOLUTE_CODEBASE_PATH}` |
| **Date** | {YYYY-MM-DD HH:MM:SS} |
| **Standard document** | `{ABSOLUTE_PATH_TO_TDD_MD}` |
| **Items reviewed** | {NUMBER} |
| **Verdict** | {Ready / Needs fixes / Blocked} |

## 1. Scope summary

[One paragraph (3–5 sentences). What was reviewed, what change set / plan / module it represents, and why it matters. Anchor to concrete artifacts (commit hashes, plan file paths, module names).]

**Items in scope:**

1. `{item_1}` — {one-line description}
2. `{item_2}` — {one-line description}
3. ...

## 2. Four-axis analysis (§3-2)

For each item, classify along the four decision axes. Use `Unknown` when evidence is insufficient — never guess silently.

| Item | ① Item nature | ② Change frequency | ③ Failure impact | ④ Recovery difficulty |
| --- | --- | --- | --- | --- |
| `{item_1}` | {Pure logic / external dep / UI / data / infra} | {Low / Medium / High / Unknown} | {Low / Medium / High / Critical} | {Easy / Hard / Irrecoverable / Unknown} |
| `{item_2}` | ... | ... | ... | ... |

**Axis-driven observations:**

- {Notable patterns — e.g., "All payment-related items are High failure impact + Irrecoverable, indicating heavy prior-validation requirements."}

## 3. Decision-tree application (§3-3, §3-4)

### 3.1 Step 1 — base test set per item (§3-3)

| Item | Item nature | Base set | Tests it implies |
| --- | --- | --- | --- |
| `{item_1}` | {nature} | {A/B/C/D/E or union} | {required tests} |
| `{item_2}` | ... | ... | ... |

### 3.2 Step 2 — risk augmentation per item (§3-4)

For each item, mark Yes/No for the six risk questions and list the additions.

| Item | R1 Money/legal | R2 Traffic | R3 Migration | R4 Data loss | R5 Devices | R6 UX/conversion | Added tests |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `{item_1}` | Yes | No | No | Yes | Yes | No | + Security, Mutation, Recovery, Idempotency, Compatibility |
| `{item_2}` | ... | ... | ... | ... | ... | ... | ... |

### 3.3 Final required test set per item

```mermaid
flowchart LR
    Item1[`{item_1}`] --> S1[Unit · Integration · Security · Mutation · Compatibility · Regression]
    Item2[`{item_2}`] --> S2[...]
```

(Replace with the actual items and their required test sets. Keep it Mermaid.)

## 4. Matrix cross-check (§3-5)

Compare each item against its row in the §3-5 matrix. Mark **Required (●)** vs **Recommended (●)** vs **Situational (△)** vs **Not needed (—)** on the *required* side, then report **Present / Missing** on the actual side.

| Item | Test type | Required by §3-5? | Currently present? | Gap class |
| --- | --- | --- | --- | --- |
| `{item_1}` | Unit | ● Required | Present (`tests/test_lockout.py`) | OK |
| `{item_1}` | Mutation | ● Recommended | Missing | Important |
| `{item_1}` | Security | ● Required | Missing | **Critical** |
| `{item_2}` | ... | ... | ... | ... |

## 5. Sufficiency diagnosis (§2-1, §2-2, §2-3)

Independent of category, evaluate the *quality* of the existing test design.

| Lens | Finding | Evidence | Severity |
| --- | --- | --- | --- |
| Code coverage (§2-1) | {e.g., No coverage tool configured / coverage is N% / branch X uncovered} | {`pyproject.toml` lacks `coverage`; PR diff at `auth/lockout.py:42` introduces an `if` with no test} | Important |
| Mutation testing (§2-2) | {e.g., No mutation runs on critical-logic modules} | {`auth/`, `payment/` have no PIT/Stryker/mutmut config} | Critical (for high-risk logic) |
| Edge / boundary (§2-3) | {e.g., Boundary inputs untested in `lockout`} | {No `test_*_boundary_*` or equivalent in `tests/test_lockout.py`} | Important |

## 6. Gaps and recommendations

### 6.1 Critical (must fix before ship)

[Required test types missing in high-impact / low-recoverability items.]

1. **{Gap title}**
   - **Item:** `{item}`
   - **Missing:** {test type, e.g., security tests for login}
   - **Why it matters:** {impact + cite §, e.g., "Login is High failure impact + Irrecoverable per §3-2; §3-4 R1 requires security and mutation tests."}
   - **Recommendation:** {concrete action with file paths, e.g., "Add SQL-injection and CSRF tests under `tests/security/test_login.py`. Add mutation test for `auth/lockout.py:check_threshold` boundary."}

_None found._ (delete if there are issues)

### 6.2 Important (should fix soon)

[Recommended test types missing, or required types present but appear to skip key cases.]

1. **{Gap title}**
   - **Item:** `{item}`
   - **Missing / weak:** {description}
   - **Why it matters:** {cite §}
   - **Recommendation:** {concrete action}

_None found._ (delete if there are issues)

### 6.3 Minor (nice to have)

[Situational tests, structural / configuration improvements.]

1. **{Gap title}**
   - **Item:** `{item}`
   - **Missing / weak:** {description}
   - **Why:** {cite §}

_None found._ (delete if there are issues)

## 7. Strengths

[What the existing test design does well. Be specific with file:line references — these are signals to preserve under refactoring.]

1. **{Strength title}** — {description} (`file:line`)

## 8. Top 3 actions (executive summary)

The three actions a dispatcher should take next, in priority order. Each must be runnable.

1. {Action with TDD.md section reference, e.g., "Add a mutation test for `auth/lockout.py:check_threshold` covering boundary `>= 5` (§2-2, §3-4 R1)."}
2. {Action with TDD.md section reference}
3. {Action with TDD.md section reference}

## 9. Verdict and next action

**Verdict:** {Ready / Needs fixes / Blocked}

**Reasoning:** {1–2 sentences justifying the verdict using gap counts and the §3-1 risk lens.}

**Gap counts:** {N} Critical · {N} Important · {N} Minor

**Recommended next action for the dispatcher:**

- {e.g., "Halt implementation and address the Critical gaps before the next subagent runs."}
- {e.g., "Proceed with implementation but file tickets for the Important gaps before merge."}
- {e.g., "Proceed; revisit Minor items during the next refactor."}
