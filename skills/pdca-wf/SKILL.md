---
name: pdca-wf
description: Use when building one feature/task end-to-end with deterministic Workflow execution and verify-to-100 — i.e. a single PDCA cycle where Research/Do/Check are run as native Workflow scripts while planning stays in the main session. Triggers on "pdca workflow", "run this feature with workflow", "build + verify to 100", single-feature deterministic build. NOT for multi-feature initiatives (use cowork-sprint).
---

# pdca-wf — PDCA as a native Workflow execution engine

## One-line maxim

**Judgment (Plan·Design) stays in main with thinking; structured/bulk/parallel execution (Research·Do·Check) runs as Workflow scripts that drive to 100 — only the irreversible launch is gated back to main.**

Single input for this skill's own design: `docs/02-planned/*-pdca-wf-design.md` (read it if extending the skill).

## Why this shape (hard constraints — do not fight these)

- Workflow agents have **thinking OFF** → Plan/Design (judgment) MUST stay in main, never inside a Workflow.
- Workflow nesting is **1-level** → this skill never nests; cowork-sprint calls it, it does not call itself.
- Workflow runtime is a **sandbox, NOT Node**: `Date.now()`/`new Date()`/`Math.random()` throw; no `fs`. → main stamps timestamps via `date` and injects them through `args`.
- Therefore: **phase boundary = main re-entry point.** Main runs each execution phase as a Workflow, reads the structured result, then decides the next phase.

## Three actors

| Actor | Owns |
|---|---|
| **Main** | judgment: Plan·Design, phase-boundary decisions, irreversible gate, timestamp stamping |
| **Script** | wiring: which agents run in what order/parallelism, when to loop/stop (deterministic, no thinking) |
| **Agent** | work: actual read/write/search/verify |

## Agent lifecycle (discover → reuse / create → use → evolve) — main-owned

Workflow `agent()` accepts `agentType` and resolves it from the **same registry as the Agent tool** (project `.claude/agents/` → user `~/.claude/agents/` → plugins). So scripts can dispatch purpose-fit agents — but selection/creation/evolution judgment stays in main (Workflow sandbox cannot Write files).

| Step | Who | How |
|---|---|---|
| **Discover** | main (may dispatch an Explore agent) | before Do/Check, scan agent registries for a fit |
| **Reuse** (fit exists) | script | `agent(prompt, {agentType:'<name>', schema})` |
| **Create** (no fit + repeatable) | **main** | Write a project-local `.claude/agents/<name>.md` (clear role, least-privilege tools), THEN script uses it via `agentType`. Scripts have no `fs` — they can never create agents. |
| **Don't create** (one-off / near-duplicate) | script | default workflow agent (omit `agentType`) |
| **Evolve** (after a phase) | **main** | read structured results (gaps, residuals, which agent underperformed) → Edit that agent `.md` to improve it. Self-evolution loops in main, never in the sandbox. |

This is the create-vs-reuse gate (agent-skill-authoring rule): create when role absent + repeatable; reuse when discovered agent approximately fits (then evolve); don't create for one-off inline.

## Procedure (each phase = one TodoWrite item; do not skip ahead)

Create a TodoWrite todo per phase. Mark `in_progress` on entry, `completed` only when its exit condition holds.

### Phase 0 — Stamp + scope (main)
1. Run `date '+%Y-%m-%d-%H%M'` → `<dt>`. Pick `<feature>` slug.
2. Confirm this is a **single** feature. Multi-feature → stop, route to `/cowork-sprint`.
3. Ensure target `docs/` exists with cowork-doc-sync taxonomy folders as needed.
4. **Entry-mode check — pre-planned vs interactive.** If the caller (a user pointing at an existing design doc, or `/cowork-sprint` PHASE 1) supplies a **design doc that already contains the WorkList**, this is **execution-only mode**: validate the doc has WorkList + (agentMap | derivable) + verifyCmd decision, derive `fileGroups` (Phase 3 step), then **jump directly to Phase 4**. Phases 1–3 run ONLY in interactive mode (no design doc supplied). Rationale: planning is interactive; re-running it mid-autonomous-flow would pause the caller — pre-planned input means planning already happened.

### Phase 1 — Research (Workflow)
- Invoke `Workflow({script, args:{feature, dt}})` using the Research template in `references/workflow-scripts.md`.
- Script fans out a multi-modal sweep (code / web / entity), returns `ResearchFindings` (schema in `references/schemas.md`).
- Main writes findings → `06-research/<dt>-<feature>.md`.
- Exit: findings file exists.

### Phase 2 — Plan (MAIN, thinking)
- Read findings. Design the approach in the main session (thinking active — never delegate).
- Write `02-planned/<dt>-<feature>-plan.md` (status ACTIVE-PLAN).
- Exit: plan file exists.

### Phase 3 — Design (MAIN, thinking) — 박제
- Converge the design into `02-planned/<dt>-<feature>-design.md` (status ACTIVE-PLAN). **This doc is the single input to Do.**
- Produce a `WorkList` **as a JSON value** (schema in `references/schemas.md`) held in session AND embedded in the design doc for humans. Items: `{id, file, change, dependsOn}`.
- **Main pre-processes the WorkList before Do**: topo-sort by `dependsOn`; build `fileGroups` (one array per file, dependency-ordered) so same-file items serialize and disjoint files parallelize.
- Build `agentMap` `{[itemId]: agentType, fix: agentType}` from the agent-lifecycle step (discover/reuse/create). Omit entries to use the default workflow agent.
- Detect `verifyCmd` for this stack (e.g. `npm test && npm run lint && tsc --noEmit`), or `null` if non-verifiable.
- Exit: design doc + WorkList(JSON) + fileGroups + agentMap + verifyCmd ready.

### Phase 4 — Do (Workflow)
- Invoke `Workflow({script, args:{workList, fileGroups, agentMap, designPath, dt, feature}})` using the Do template. **Inline the real schemas into the script string** (sandbox has no fs).
- Script runs `fileGroups` with `parallel()` across files and serial within a file (no per-item worktree — same-file serialization prevents lost-update).
- Exit: Workflow returns built result.

### Phase 5 — Check/Act (Workflow, loop-to-100)
- Invoke `Workflow({script, args:{designPath, verifyCmd, agentMap, dt, feature}})` using the Check template (schemas inlined).
- **Verifiable work (`verifyCmd` set): the script RUNS the real stack checks first** (exit code, not opinion); matchRate==100 requires executed checks green AND lenses==100. Non-verifiable: lenses only, ≥90 floor.
- Lenses = perspective-diverse verify (correctness / regression / design-fit) vs the design doc; gaps fixed with `parallel()`, loop until 100 or max 5.
- Main re-stamps `date` and writes `05-reports/<dt2>-<feature>-check.md` (the check snapshot uses its OWN datetime, not Phase-0 `<dt>`).
- **Quality gate = NO BRANCH**: do not pause on matchRate<100. If max 5 exhausted < 100 → carry residualGaps to Report. Do not stop.
- Exit: GapResult returned (100 or residual recorded).

### Phase 6 — Report + lifecycle (main)
- Re-stamp `date` → `<dt3>`. Write `05-reports/<dt3>-<feature>-report.md` (phaseHistory passed through from Check's returned `iterations`/`testsRun`, NOT LLM-reconstructed; matchRate, residualGaps, carryItems).
- Update `01-built/<feature>.md` (LIVING, as-built). See **Document lifecycle** below.
- Hand off to `/cowork-doc-sync` for final taxonomy alignment.
- Exit: report + 01-built updated, planned reconciled.

## Gate model (two orthogonal axes)

| Gate | Axis | Handling |
|---|---|---|
| No-Go / matchRate<100 | quality | **no branch** — loop-to-100, on miss record residual gaps in report, keep going |
| git push · deploy · vault bulk · remote migration | safety | **gate stays** — Workflow does everything up to it; the actual launch is approved in main |

Quality is solved by score; safety is NOT (irreversible even at 100). Never auto-fire irreversible actions inside a Workflow.

Verifiable work targets 100%. Non-verifiable work floors at ≥90% (CLAUDE.md PDCA rule).

## Document lifecycle (on build completion)

**DONE predicate (code-checkable, gates the irreversible delete — NOT the raw LLM float):**
`done := every WorkList item present in built result AND no blocker/major gaps in GapResult`. Compute in main. Only `done` triggers design-doc deletion; a hallucinated `matchRate:100` without item-coverage does NOT. **`done` always overrides the ≥90 floor for the delete decision** — non-verifiable work at matchRate 92 with an open `major` gap is `done==false` → KEEP the design doc.

```
done == true (all built):
  01-built/<feature>.md            ← as-built, CLEAN. Section-scoped MERGE: replace only the sections THIS cycle changed; never wipe the whole file (it may cover other features). Old superseded sections are deleted (not struck); git holds history.
  02-planned/<dt>-<feature>-design.md  ← DELETE the file (no strikethrough on a doomed file).
  02-planned/<dt>-<feature>-plan.md    ← also superseded → cowork-doc-sync deletes / moves to 04-legacy.

done == false (residual after max 5):
  01-built/<feature>.md            ← as-built of implemented parts only, CLEAN (section-scoped merge).
  02-planned/<dt>-<feature>-design.md  ← KEEP. Strike through implemented items; un-struck = residual.
                                          **≥50% of items struck AND struck count ≥3 → DELETE the struck
                                          items instead** (keep only residual + one line "implemented →
                                          01-built / git"). Small-doc guard: with <3 struck items the ratio
                                          trips too easily (1 of 2 = 50%) — keep strikethrough there.
                                          100% struck = done case above → delete the file.
  02-planned/<dt>-<feature>-plan.md    ← KEEP (still active).
  05-reports/<dt3>-<feature>-report.md ← residual gap list (re-pursue later as a NEW dated 02-planned plan).
```

Rules:
- **01-built never has strikethrough**, and edits are **section-scoped merges** — replace only this cycle's sections, never whole-file overwrite (protects the single-LIVING authority when one file spans multiple features).
- **Strikethrough lives only in 02-planned** (residual case), marking planned items that got built — cancellation marker, not preservation, kept short/inline. **Noise cap**: once struck items reach ≥50% of the plan AND there are ≥3 of them, delete them (leave residual + a one-line pointer); a doc that is mostly strikethrough misleads more than it informs, and git holds the history. Small-doc guard: below 3 struck items the ratio is meaningless — keep the strikethrough.
- **Deletion is terminal + idempotent.** The design doc must persist through Check; delete only here in Phase 6. **Resume guard**: if the design doc is absent AND `01-built/<feature>.md` exists → the cycle is already complete; do NOT re-run Check.
- Re-pursuing a residual → fresh dated `02-planned` plan (history separated by datetime).

## Structured output (code consumes LLM output → schema-forced, never free-text+regex)

All execution-phase scripts return schema-validated JSON. Full schemas: `references/schemas.md`.
`ResearchFindings` · `WorkList` · `GapResult{matchRate,gaps[]}` · `Report{phaseHistory,matchRate,residualGaps,carryItems}`.

## Red flags — STOP

- About to put Plan/Design logic inside a Workflow script → STOP (thinking is off there; keep it in main).
- About to call `Date.now()`/`new Date()` in a script → STOP (it throws; stamp in main, pass via args).
- About to pause the Workflow on matchRate<100 → STOP (quality gate has no branch; loop-to-100 then report).
- About to auto-run git push / deploy / vault bulk inside a Workflow → STOP (safety gate stays in main).
- Multi-feature scope creeping in → STOP, route to `/cowork-sprint`.

## Quick reference

| Phase | Actor | Output | Gate |
|---|---|---|---|
| 0 Stamp | main | `<dt>`, scope check | single-feature only |
| 1 Research | Workflow | `06-research/<dt>-<feature>.md` | — |
| 2 Plan | main (thinking) | `02-planned/<dt>-<feature>-plan.md` | — |
| 3 Design | main (thinking) | `02-planned/<dt>-<feature>-design.md` + WorkList | 박제 = Do input |
| 4 Do | Workflow | code | — |
| 5 Check/Act | Workflow loop-to-100 | `05-reports/<dt2>-<feature>-check.md` | quality: no branch |
| 6 Report | main | report + `01-built/<feature>.md` + cowork-doc-sync | safety: irreversible gated |

References: `references/workflow-scripts.md` (script templates) · `references/schemas.md` (JSON schemas) · `references/taxonomy-map.md` (taxonomy + lifecycle + cowork-doc-sync handoff).
