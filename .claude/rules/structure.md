# System Structure

## Data Files

Eight JSON files in `my-data/` track the user's personal information. Schemas are defined in `schemas/` and documented in the Data Routing rules.

- `health-profile.json` — identity, conditions, goals, providers, medications, family history
- `status.json` — quick context summary, current priorities, pending action items, onboarding state
- `lab-results.json` — timestamped results with context, interpretation, next recheck dates
- `symptoms.json` — observations with severity, triggers, pattern analysis, hypotheses
- `treatments.json` — medications, supplements, therapies, environmental modifications
- `lifestyle.json` — sleep, diet, stress, exercise, lifestyle factors
- `visit-notes.json` — visit summaries, care team, orders, follow-ups
- `decisions.json` — permanent decision log with reasoning (never pruned, reversals reference originals)

User preferences: `preferences.json` — overrides for system defaults (severity scale, communication preferences, etc.)

## Knowledge Files

**Reference** (`reference/`) — interpreted synthesis. Contains pointers to source data (my-data/) and evidence (research/), plus interpretations of what the combination means for this person. Never contains copies of raw values or evidence. Read at session start via `reference/INDEX.json`. For treatment discussions, provider prep, trend assessment.

**Research** (`research/`) — deep, mechanism-level, cited evidence. Each file has YAML frontmatter with `date_created`, `topic`, and `entities` for cross-referencing. Read on demand via `research/INDEX.json`. For "why" questions, claim verification, novel situations.

**Entity index**: both INDEX files include an `entities` field on each file entry, listing the canonical entity names (conditions, medications, lab markers, mechanisms, interactions, symptoms, providers) covered by that file. The entity index enables:
- **Cross-referencing**: find all files that discuss a specific entity
- **Gap detection**: identify medications, conditions, or markers with no research coverage
- **Propagation**: when source data changes, find which reference files to flag for review
- **Tiered loading**: in large knowledge bases, load only files relevant to the current session

When citing reference files, point to the underlying research file for full evidence. When updating reference files, follow the pointer+interpretation format in `guides/research-guide.md` (Phase 4).

## Schemas

Formal JSON Schema definitions in `schemas/`. One file per data type. These define the expected shape of each data file and serve as the canonical reference for field names, types, and descriptions. The LLM reads these schemas via the Data Routing rules. Future tools may use the schema files directly for validation and interoperability.
