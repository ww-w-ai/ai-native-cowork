> Status: LIVING — as-built. Live authority = `../../SKILL.md` + `../../references/*`. This is the consolidated as-built summary (supersedes the 02-planned design docs).
> Last updated: 2026-06-06 13:05

# pdca-wf — as-built

Single-feature PDCA cycle with native Workflow as the execution engine. Lives in the ai-native-cowork plugin (moved from user-local to resolve the plugin→user dependency direction).

## What was built (authority pointers)

- **Orchestration**: `SKILL.md` — 6 phases (Research·Plan·Design·Do·Check·Report). Main owns Plan/Design (thinking); Research/Do/Check run as native Workflow scripts.
- **Workflow script templates**: `references/workflow-scripts.md` — runtime contract (`export const meta` + ambient `agent/parallel/pipeline/phase/log/args`; top-level await/return OK; sandbox forbids Date/fs; schemas inlined by main).
- **Schemas**: `references/schemas.md` — ResearchFindings, WorkList(+fileGroups), agentMap, GapResult, Report.
- **Taxonomy + lifecycle**: `references/taxonomy-map.md` — cowork-doc-sync taxonomy outputs, datetime filenames, document lifecycle (01-built clean/section-merge; 02-planned strikethrough with noise cap: ≥50% struck AND ≥3 struck → delete struck items, <3 struck = keep), cowork-doc-sync handoff.
- **Doc output templates**: `references/doc-templates.md` — fixed fill-in skeletons for Plan/Design(+WorkList)/Check/Report(+QA table, anticipated questions)/01-built section. Filled, never restructured.

## Key decisions (as-built)

- **Three actors**: main = judgment; script = wiring; agent = work.
- **Gate model (2 axes)**: quality (matchRate) = NO branch, loop-to-100 (max 5), miss → post-hoc report. Irreversible (push/deploy/vault) = gate stays in main (structural via least-privilege agents); before the launch main runs a thinking adversarial review (Check lenses are thinking-off, so main does the one judgment pass), then approves.
- **Verify-to-100 grounded**: for verifiable work the Check script RUNS the stack's real checks (verifyCmd); 100 requires executed-green AND lenses==100. Non-verifiable floors at ≥90.
- **DONE predicate** (code-checkable: all WorkList items present + no blocker/major gaps) gates the irreversible design-doc delete — not the raw LLM float.
- **Agent lifecycle**: discover/reuse via `agentType`; create/evolve in main (sandbox can't write files).
- **Entry modes (Phase 0)**: interactive (no design doc → Phases 1-3 dialogue) vs **execution-only** (design doc with WorkList supplied — by user or cowork-sprint PHASE 1 — validate then jump to Phase 4; planning is never re-entered mid-autonomous-run).
- **Trigger surface**: fires on implicit single-feature build requests ("build me this feature", "implement X") — not only explicit skill-name mentions. NOT for multi-feature (cowork-sprint), <~30min trivial edits, pure Q&A.
- **standalone — no external plugin dependency.**

## Quality history

RED→GREEN→REFACTOR (writing-skills): adversarial + application tests found 9 blockers/majors (script contract, inlined schemas, designPath/agentMap wiring, dependsOn topo, worktree serialization, lifecycle invariant, verify-to-100 grounding, timestamp restamp) — all resolved; GREEN re-audit confirmed; 4 follow-up issues (dt2/dt3 threading, DONE-vs-floor, fileGroups $ref, phaseHistory pass-through) fixed.

## Related

Retrospective phase (repo-local only, user-gated) was designed alongside and implemented in `../../cowork-sprint/SKILL.md` PHASE 2 — re-centered on SELF-EVOLUTION after the first 12-sprint field run (A agent-evolution + B self-assessment with [FIXABLE-PROMPT|FIXABLE-SCRIPT|PROCESS] tags + C rule-promotion + E carry; fixed template `cowork-sprint/templates/retrospective.template.md` ending in a numbered APPLY-GATE table). Same field run added: per-sprint QA TABLE gate (Leader-built feature→check table; unchecked row without deferral reason = FAIL), INTEGRATION common-extraction pass after parallel fan-out, compact-return schema for verification panels, and `templates/sprint-report.template.md`. Mid-cycle agent evolution stays unblock-only.
