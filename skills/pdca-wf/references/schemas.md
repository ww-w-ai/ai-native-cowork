# pdca-wf — structured output schemas

All execution-phase Workflow scripts MUST return schema-validated JSON via `agent(prompt, {schema})`. Never free-text + regex parsing (CLAUDE.md structured-output rule). `schema` is a raw JSON Schema object.

**Sandbox note:** scripts have no `fs` — they cannot read this file at runtime. **Main inlines the real schema object into the script string** before `Workflow({script})`. The templates' `{ /* ... */ }` placeholders are where main pastes these.

## ResearchFindings (Phase 1)

```json
{
  "type": "object",
  "required": ["findings"],
  "properties": {
    "findings": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["area", "claim", "confidence"],
        "properties": {
          "area":       { "type": "string", "description": "code | web | entity | constraint" },
          "claim":      { "type": "string" },
          "source":     { "type": "string", "description": "file:line, URL, or entity id" },
          "confidence": { "type": "string", "enum": ["high", "medium", "low"] }
        }
      }
    }
  }
}
```

## WorkList (Phase 3, produced by MAIN, consumed by Do)

```json
{
  "type": "object",
  "required": ["items"],
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id", "file", "change"],
        "properties": {
          "id":        { "type": "string" },
          "file":      { "type": "string" },
          "change":    { "type": "string", "description": "what to implement" },
          "dependsOn": { "type": "array", "items": { "type": "string" } }
        }
      }
    },
    "fileGroups": {
      "type": "array",
      "description": "built by main: one inner array per file, items in dependency order (each inner element is a WorkList item — same shape as items[]). Do runs groups in parallel, items within a group serially (same-file = no lost-update).",
      "items": { "type": "array", "items": { "$ref": "#/properties/items/items" } }
    }
  }
}
```

## agentMap (Phase 3, produced by MAIN, consumed by Do/Check)

```json
{
  "type": "object",
  "description": "work-item id → agentType (reuse/created agent). 'fix' key = agentType for Act-step gap fixes. Omit any entry to use the default workflow agent.",
  "properties": { "fix": { "type": "string" } },
  "additionalProperties": { "type": "string" }
}
```

`verifyCmd` (Phase 3, consumed by Check) is a plain string (e.g. `"npm test && npm run lint && tsc --noEmit"`) or `null` for non-verifiable work — not a schema'd agent return.

## GapResult (Phase 5)

```json
{
  "type": "object",
  "required": ["matchRate", "gaps"],
  "properties": {
    "matchRate": { "type": "number", "minimum": 0, "maximum": 100 },
    "gaps": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["item", "expected", "actual", "severity"],
        "properties": {
          "item":     { "type": "string", "description": "WorkList id or design section" },
          "expected": { "type": "string" },
          "actual":   { "type": "string" },
          "severity": { "type": "string", "enum": ["blocker", "major", "minor"] }
        }
      }
    }
  }
}
```

## Report (Phase 6)

```json
{
  "type": "object",
  "required": ["matchRate", "phaseHistory", "residualGaps", "carryItems"],
  "properties": {
    "matchRate":    { "type": "number" },
    "phaseHistory": { "type": "array", "items": { "type": "object" } },
    "residualGaps": { "type": "array", "items": { "type": "object" }, "description": "GapResult.gaps unresolved after max 5" },
    "carryItems":   { "type": "array", "items": { "type": "string" }, "description": "re-pursue as new dated 02-planned plan" }
  }
}
```

## Merge note

When verify runs N lenses in `parallel()`, merge in code (not via an agent): `matchRate = min(lensRates)` (conservative — a gap any lens finds counts), `gaps = dedupe(concat(all lens gaps))`. Keep merge deterministic so resume is stable.
