# cowork-doc-sync convention — taxonomy + status + migration (single-authority spec)

> Both `cowork-doc-sync` and `cowork-doc-init` read this file as the target spec.
> Organization standard for a project repo's `docs/`.

## 1. Folder taxonomy

Numbering applies **only to docs we authored.** Tool-generated artifacts (§4) are outside the numbering.

| Folder | Content | Default status |
|---|---|---|
| `00-reference` | Curated stable background (product definition, naming, external-system analysis, porting specs) | LIVING/stable |
| `01-built` | **Implementation/current — single LIVING authority** (as-built architecture). "The current truth is here" | **LIVING** |
| `02-planned` | Plan/future (decided but not yet built) | ACTIVE-PLAN |
| `03-manual` | User/technical manuals + handover | LIVING (synced) |
| `04-legacy` | Superseded/deprecated docs (once authoritative) | FROZEN |
| `05-reports` | Work snapshots: PDCA, gap analysis, code review, code-health observations, session reports. Immutable, by date | FROZEN-by-nature |
| `06-research` | Technical/domain research = decision evidence (CF/library/platform investigation). By date | FROZEN-by-nature |
| `99-misc` | Unclassified catch-all inbox (temporary, hard to classify) | temporary |

File convention: `05-reports`/`06-research` use a date prefix `YYYYMMDD-<topic>.md` (chronological sort).

## 2. Status model (lifecycle ≠ folder, an overlay)

| Status | Meaning | Synced? |
|---|---|---|
| **LIVING** | Current truth. The LLM treats only this as "true now" | ✅ always |
| **ACTIVE-PLAN** | Decided but not yet built. Once built, folded into LIVING then FROZEN | once when built |
| **FROZEN** | Point-in-time record. Not the current truth | ❌ never |

- **Single LIVING authority = the as-built docs in `01-built` (+ the root CLAUDE.md summary).** Enforces "if you want the current state, look only here."
- Recommend a 1-line status label at the top of every doc: `> Status: LIVING | ACTIVE-PLAN | FROZEN (date) — SUPERSEDED by <link>`.

## 3. Migration rules (goal = prevent LLM misdirection + preserve history)

**Core: git is the history layer.** Anything moved or deleted is fully recoverable via git → no need for "strikethrough to preserve."

| Situation | Handling |
|---|---|
| Whole doc superseded | **Move to `04-legacy` + tombstone header** (link to current truth). Not strikethrough/delete |
| Move a section to another doc | **Verbatim move** (no rewrite/summary). Original location is **deleted** (default) + if needed a 1-line pointer `(→ current location is <location>)` |
| Pure clutter (zero value, only confusion) | **Delete** |
| Strikethrough (`~~~~`) | **Narrow exception only** — when there is an educational reason to show "the process of changing one's mind" in place, keep it short. (Strikethrough text is still read by the LLM, so it is useless for preventing misdirection → minimize) |

> Why minimize strikethrough: markdown strikethrough is still context tokens → the LLM cannot reliably discount it. Since git + 04-legacy capture history, keep the body clean (delete) = LLM safety.

**Update inbound references when moving a doc (MUST — easy to miss):** moving a doc **breaks the path links in other docs that pointed to it** (the tombstone is only at the new location; the old-path inbound becomes a 404). After a move:
- **LIVING/ACTIVE/manual → MOVED links** must be updated. If it was "for the current state, see this plan" → **redirect to LIVING (as-built)** (not the plan). If it is a plain path reference → the new path (04-legacy/05-reports/…).
- **intra-legacy/frozen internal links** (both moved together into frozen) = **preserve** (history, do not over-maintain). Even if broken it is fine, since it is inside a frozen doc.

**fold-before-move reinforcement — "LIVING cites it as detail = not-folded":** even for a completed plan, if a **LIVING doc cites that plan as the "detail spec"** → the current truth is only *summarized* in LIVING and the detail lives only in the plan = **not fully-folded → must not legacy it.** Keep in place with a FROZEN-built label. (Legacy only when LIVING has absorbed the detail too.)

## 4. External/tool-generated artifacts (outside our taxonomy — not controllable)

| Source | Handling |
|---|---|
| `commit-log/` (cowork-commit recap) | Keep and commit. Valuable. **Not numbered** |
| other tools' generated scratch dirs (e.g. numbered plan/design dumps, tool state dirs) | **.gitignore**. Not tracked. Ignore |
| Other tools | Noise = gitignore, valuable = leave as-is but **do not absorb** into 00-99 |

> Principle: do not try to pull tool output into our taxonomy (a losing battle). If the name collides (`01-built` vs `01-plan`), disambiguate + this one-line rule is enough.

## 5. vault vs repo boundary (aligned with the global rule)

| Artifact | Location |
|---|---|
| Technical/platform/domain research (CF, library, architecture investigation) | repo `06-research` (engineering) |
| Product/business/market research (persona, beachhead, market size, discovery) | **vault (www-wiki) summary** — not the repo |
| Code implementation plan, design, refactoring plan | repo `02-planned` |

Criterion: "is it product/business knowledge (→vault) vs engineering (→repo docs/)."

## 6. Local project config contract (the standard each repo instantiates)

This file is the **generic default**. Each repo declares its **project-specific** doc/folder management in a **local doc-sync config** — either `docs/CONVENTION.md` (formal hook) **or** a `## doc-sync 범위` (doc-sync scope) section in the repo's `CLAUDE.md`/`AGENTS.md`. The skill reads it **every run**; if absent, offer to scaffold one from this contract. The generic method stays in the skill; the per-project *what-not-to-miss* lives here.

A conforming local config declares (fill what applies; omit where it inherits the default):

1. **Taxonomy deviations** — non-default folders or numbering vs §1 (none = inherits §1).
2. **LIVING authority** — the file(s) that are the single current truth (default: `01-built/<x>.md` + the root CLAUDE.md summary). List project additions.
3. **Sync surfaces beyond `docs/`** — when a feature ships, which OTHER surfaces must align in the **same** pass. The skill aligns `docs/`; THESE are the project's extra must-not-miss. Enumerate, e.g.: user-facing whitepapers/manuals, the repo `CLAUDE.md`/`AGENTS.md`, AI-facing guides (help corpus, tool reference), QA/checklist seeds, contract "mirror surfaces" (schemas/messages/output shapes).
4. **Status-claim verification** — docs that assert **VCS/deploy/version/release STATE** (labels like *merged / deployed / pending / unshipped / vN*) are **high-rot and invisible to content drift**. List each such label surface + HOW to verify against source-of-truth: e.g. merge = `git branch --contains <c>`, deploy = last-deploy time vs last code commit, version = the code constant, DB = `<migrate-tool> list --remote`. doc-sync MUST verify + relabel, never trust the label. *(Real failure: a "unshipped" label that was actually deployed nearly drove a redundant re-deploy.)*
5. **Derived/built docs** — any "edit source → run build → never hand-edit the derived/public copy" chains. List source → derived + the build command (e.g. whitepaper `build.sh` → `public/*.html`).
6. **vault vs repo specifics** — project tags/paths if the §5 boundary needs detail.

**Skill behavior tie-in:** the generic **status-claim drift** class (#4) is a default drift class in the workflow — docs asserting merge/deploy/version/release state are verified against source-of-truth (VCS/CI/prod), not just code content. The local config supplies the *HOW* (which commands). If a repo has no local config, the skill still applies §1–§5 defaults + flags that project-specific surfaces (#3,#4) are undeclared.
