# System Structure

## Data Files

JSON files `my-data/` track user's personal info. Schemas defined `schemas/`, documented Data Routing rules. One canonical home per fact; named entities join across files by `slug`.

- `health-profile.json` — identity, conditions, goals, providers, allergies (NOT medications/supplements, NOT family history)
- `status.json` — quick context summary, current priorities, pending action items, onboarding state
- `lab-results.json` — timestamped results (native numerics), context, interpretation, next recheck dates
- `symptoms.json` — observations with severity, triggers, pattern analysis, hypotheses
- `treatments.json` — the single home for every intervention: medications, supplements, therapies, environmental modifications
- `lifestyle.json` — baseline context: sleep, diet, stress, exercise, lifestyle factors
- `visit-notes.json` — visit summaries, care team, orders, follow-ups
- `decisions.json` — permanent decision log with reasoning (never pruned, reversals reference originals)
- `family-history.json` — the subject's family health history, one entry per relative (hereditary-risk context)
- `genomic-markers.json` — genetic variants, methylation states, pharmacogenomic phenotypes
- `training.json` — structured resistance/performance programming
- `photos.json` — progress/aspirational/documentation photos (registered by ID; files stay local)
- `strategic-plans.json` — measurement-driven plans tied to outcomes (baseline, target, monitoring, triggers)

User preferences: `preferences.json` — overrides system defaults (severity scale, communication preferences, etc.)

## Knowledge Files

**Reference** (`reference/`) — interpreted synthesis. Pointers to source data (my-data/) + evidence (research/), plus interpretations what combination means for person. Never copies raw values or evidence. Read session start via `reference/INDEX.json`. For treatment discussions, provider prep, trend assessment.

**Research** (`research/`) — deep, mechanism-level, cited evidence. Each file YAML frontmatter with `date_created`, `topic`, `entities` for cross-referencing. Read on demand via `research/INDEX.json`. For "why" questions, claim verification, novel situations.

**Entity index**: both INDEX files include `entities` field per file entry, listing canonical entity names (conditions, medications, lab markers, mechanisms, interactions, symptoms, providers) covered. Entity index enables:
- **Cross-referencing**: find all files discuss specific entity
- **Gap detection**: identify medications, conditions, markers no research coverage
- **Propagation**: source data changes → find which reference files flag for review
- **Tiered loading**: large knowledge bases, load only files relevant current session

Citing reference files → point to underlying research file for full evidence. Updating reference files → follow pointer+interpretation format `guides/research-guide.md` (Phase 4).

## Schemas

Formal JSON Schema definitions `schemas/`. One file per data type. Define expected shape each data file, serve as canonical reference field names, types, descriptions. LLM reads schemas via Data Routing rules. Future tools may use schema files directly for validation + interoperability.