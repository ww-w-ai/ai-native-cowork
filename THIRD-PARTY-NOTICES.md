# Third-Party Notices

This project (ai-native-cowork) includes material adapted from third-party
open-source projects. Their licenses and attributions are listed below.

---

## bkit (bkit-claude-code)

- **Source**: https://github.com/popup-studio-ai/bkit-claude-code
- **Author**: POPUP STUDIO PTE. LTD.
- **License**: Apache License 2.0 (permissive)

### What was adapted

The cowork-sprint **dev profile** vendors and adapts material from bkit. In all
cases the *method/approach/expertise* was adapted — bkit-plugin infrastructure
references (PDCA state, `.bkit/` store, `bkit.config.json`, `lib/`, M1-M10 SSoT,
CI invariants) were removed so the result runs standalone with **no bkit install
required**. Where verbatim agent prose was carried over, it was modified to fit
cowork's structure and philosophy.

**Vendored agents** (in `agents/`, each carries an in-file attribution header):
- `gap-detector.md` — adapted from bkit `gap-detector`
- `code-analyzer.md` — adapted from bkit `code-analyzer`
- `design-validator.md` — adapted from bkit `design-validator`
- `security-architect.md` — adapted from bkit `security-architect`
- `qa-test-planner.md` — adapted from bkit `qa-test-planner`
- `qa-test-generator.md` — adapted from bkit `qa-test-generator`
- `qa-debug-analyst.md` — adapted from bkit `qa-debug-analyst`; the docker-log /
  `zero-script-qa` skill assumption was removed and the log/trace mechanism
  generalized to be runtime-agnostic (uses whatever log surfaces the stack exposes).
- `frontend-architect.md` — adapted from bkit `frontend-architect`
- `infra-architect.md` — adapted from bkit `infra-architect`
- `enterprise-expert.md` — adapted from bkit `enterprise-expert`
- `bkend-expert.md` — adapted from bkit `bkend-expert` (targets the bkend.ai BaaS
  service; optional, project-dependent)

**Adapted methods/ideas** (in `skills/cowork-sprint/references/`, `templates/`):
- gap-analysis classification (`done/partial/missing/divergent` → matchRate) and the
  two-axis QA gate — adapted from bkit's gap-detector approach
  (`references/gap-analysis.md`).
- dev profile mechanisms — Context Anchor, sprint-master-planner topo-sort +
  bin-packing scheduler, sprint-orchestrator auto-pause + measure-then-advance,
  pdca-iterator plateau/anti-gaming, qa-lead L1-L5 taxonomy, M2/M4 design/test
  discipline (`references/dev-profile.md`).

Facts and methods are not copyrightable; no bkit source is reproduced verbatim
beyond what is noted above as adapted agent prose. Apache-2.0 requires preservation
of copyright and license notices — this file and the per-file headers satisfy that.

### Apache License 2.0 — notice

```
Copyright POPUP STUDIO PTE. LTD.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

The full Apache-2.0 license text is available at the URL above.
