# Gap-Analysis — the QA gate's second axis ("did we build all we declared?")

> Procedure for measuring `matchRate` — the QA gate's Axis 2. Generic and
> domain-agnostic. Approach adapted from **bkit gap-detector** (Apache-2.0,
> popup-studio-ai/bkit-claude-code) — method/idea only, no source text copied.
> Pairs with: PRD-lite (intent) → WorkList (declared items) → THIS (measure) →
> intent-audit (Tier-2). Read this when running the QA gate (sprint-method.md §5).

## Why two axes
The QA gate asks two orthogonal questions; both must pass for QA green:

| Axis | Question | Source |
|------|----------|--------|
| 1 — mechanical baseline | "Does it work?" | build/lint/type/test green (sprint-method.md §5) |
| 2 — gap-analysis | "Did we build all we declared?" | WorkList ↔ output (this file) |

Tests can be 100% green while half the declared work is missing — the tests cover
only the built half. Axis 1 alone back-fills a **false** matchRate. This axis
measures it for real.

## Mechanism
1. Input = the WorkList frozen in PHASE 0. Each item carries
   `{ id, description, acceptanceEvidence, priority }` (field set = local config
   knob #7).
2. For each item, compare the declared item against the actual output and classify:
   - `done` — acceptance evidence present and matches.
   - `partial` — started, evidence incomplete.
   - `missing` — no evidence.
   - `divergent` — built, but differs from what was declared.
3. `matchRate = done / total × 100`.
   - Default: **flat** (every item counts 1) — local config knob #4.
   - Override: **priority-weighted** — a missing P0 drops the rate harder than a missing P2.
4. `matchRate < threshold` (default `100`, knob #3) → feed the gap list into the
   `fix` phase and loop (cap = sprint-method.md §5b iterate cap, default 5).
   Re-classify after each fix round; never re-run blind — inject which items are
   still `missing`/`partial`/`divergent`.

## Fresh perspective (recommended)
Self-gap-analysis is biased — the executor sees its own intent, not its omissions.
Prefer a reset-context reviewer: discover a gap-detector-class agent from the
project/user/plugin pool; else dispatch `general-purpose` with "FIRST read the
WorkList + the actual outputs, THEN classify each item" as the prompt's first step.
cowork-sprint ships **no new fixed agent** for this (only `cowork-intent-auditor`
is fixed) — discovery + this procedure, per the decision in the design doc.

## Domain adaptation (no fixed layers)
- **dev**: item ↔ file/function/endpoint + its test. Optionally sanity-check one
  end-to-end data path (input→store→output) — a generalization of bkit's 7-hop
  dataFlow check, NOT the fixed 7 layers.
- **non-dev** (marketing/research/ops/data): item ↔ deliverable (a copy block, a
  report section, a dataset slice). Never hardcode dev layers into a non-code sprint.

Lenses are a local config knob (#5); default is the generic item↔evidence compare.

## Output (recorded to status.json)
- `matchRate` — the measured value (not a target).
- `gapItems[]` — `[{ id, status, note }]` from the last run; enables resume +
  the final report without recomputation.

## Dev lens (when profile: dev)
With `profile: dev` (§6A knob #9; see references/dev-profile.md §3.2), apply these
dev signals when classifying each item — they are *signals to look for*, NOT a fixed
score formula:
- **Placeholder/stub detection** → classify `partial`, never `done`: TODO/FIXME
  markers, empty handlers, skeleton bodies (`[1,2,3].map`), hardcoded sample returns.
- **3-way contract agreement** (when an API exists): spec ↔ server ↔ client must
  agree on URL/method/params/shape. Disagreement → `divergent`.
- **Anti-gaming**: evidence must be real depth, not "a file exists". Never mark
  `done` without verifying behavior; don't add comments/stubs to inflate the rate.
- **code+test pairing** (default-on at standard+ tier): an implementation item with
  no test → `partial`.
- **Dataflow sub-lens** (web-fullstack, opt-in): trace one end-to-end write path; for
  web apps the hops UI→Client→API→Validation→DB→Response→UI are an *example*
  checklist, not fixed layers.

bkit's fixed 6-axis weighted formula is an *example* a repo may adopt via config —
not hardcoded here.

## Local config
matchRate threshold (#3), method flat/weighted (#4), lenses (#5), QA-axis
enforcement (#6), and WorkList fields (#7) are read from the repo's sprint config
(`docs/CONVENTION.md` or a `## cowork-sprint scope` section in `CLAUDE.md`/`AGENTS.md`)
**every run**; omitted keys inherit the defaults above. See SKILL.md / sprint-method.md §6A.

## Provenance
The classification scheme (`done/partial/missing/divergent` → `matchRate`) and the
two-axis gate idea are adapted from bkit's `gap-detector` agent (Apache-2.0).
Facts and methods only — not copyrightable expression; no bkit source is reproduced.
