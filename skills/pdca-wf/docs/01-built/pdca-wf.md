> 상태: LIVING — as-built. Live authority = `../../SKILL.md` + `../../references/*`. This is the consolidated as-built summary (supersedes the 02-planned design docs).
> 최종 갱신: 2026-06-05 16:42

# pdca-wf — as-built

Single-feature PDCA cycle with native Workflow as the execution engine. Lives in the ai-native-cowork plugin (moved from user-local to resolve the plugin→user dependency direction).

## What was built (authority pointers)

- **Orchestration**: `SKILL.md` — 6 phases (Research·Plan·Design·Do·Check·Report). Main owns Plan/Design (thinking); Research/Do/Check run as native Workflow scripts.
- **Workflow script templates**: `references/workflow-scripts.md` — runtime contract (`export const meta` + ambient `agent/parallel/pipeline/phase/log/args`; top-level await/return OK; sandbox forbids Date/fs; schemas inlined by main).
- **Schemas**: `references/schemas.md` — ResearchFindings, WorkList(+fileGroups), agentMap, GapResult, Report.
- **Taxonomy + lifecycle**: `references/taxonomy-map.md` — cowork-doc-sync taxonomy outputs, datetime filenames, document lifecycle (01-built clean/section-merge, 02-planned strikethrough+delete), cowork-doc-sync handoff.

## Key decisions (as-built)

- **Three actors**: main = judgment; script = wiring; agent = work.
- **Gate model (2 axes)**: quality (matchRate) = NO branch, loop-to-100 (max 5), miss → post-hoc report. Irreversible (push/deploy/vault) = gate stays in main (structural via least-privilege agents).
- **Verify-to-100 grounded**: for verifiable work the Check script RUNS the stack's real checks (verifyCmd); 100 requires executed-green AND lenses==100. Non-verifiable floors at ≥90.
- **DONE predicate** (code-checkable: all WorkList items present + no blocker/major gaps) gates the irreversible design-doc delete — not the raw LLM float.
- **Agent lifecycle**: discover/reuse via `agentType`; create/evolve in main (sandbox can't write files).
- **standalone — no external plugin dependency.**

## Quality history

RED→GREEN→REFACTOR (writing-skills): adversarial + application tests found 9 blockers/majors (script contract, inlined schemas, designPath/agentMap wiring, dependsOn topo, worktree serialization, lifecycle invariant, verify-to-100 grounding, timestamp restamp) — all resolved; GREEN re-audit confirmed; 4 follow-up issues (dt2/dt3 threading, DONE-vs-floor, fileGroups $ref, phaseHistory pass-through) fixed.

## Related

Retrospective phase (A+C+E, repo-local only, user-gated) was designed alongside and implemented in `../../cowork-sprint/SKILL.md` PHASE 2 (mid-cycle agent evolution narrowed to unblock-only). See `cowork-sprint/SKILL.md`.
