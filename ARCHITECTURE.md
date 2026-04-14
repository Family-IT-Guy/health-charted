# Architecture

How Health Charted is built and why. This document is for contributors, developers, and anyone who wants to understand the system well enough to modify it or build something similar.

The system is organized around five design objectives. Each one drives specific decisions about how data is stored, how knowledge evolves, and how the user interacts with the system. Where alternatives were considered and rejected, the reasoning is included so future contributors don't re-propose solutions that have already been evaluated.

## Designed to last decades

Health data accumulated over years is irreplaceable. A system that stores a decade of lab results, treatment decisions, and research needs to be readable and functional in 10, 20, or 30 years. Every architectural choice is evaluated against this constraint.

### Plain text files, no infrastructure

All data is stored in JSON and markdown files in a folder on the user's computer. There is no database, no server, no cloud service, no account. The system requires only an AI that can read and write local files.

JSON was chosen for structured data (labs, medications, symptoms) because it enforces consistent structure, is parseable by any programming language, and is human-readable in a text editor. Markdown was chosen for prose content (research files, guides, reference summaries) because it's the most widely supported plain text format and renders well everywhere.

Nothing can go offline, get deprecated, change its pricing, or shut down. The files are the system.

### Additive-only schema changes

Data file schemas (defined in `schemas/`) follow one rule: fields can be added but never renamed or removed. A file written in 2026 must still be valid in 2036. When a new version adds fields, the AI adds them with null values the next time it writes to that file. No migration scripts, no breaking changes, no data conversion.

### Platform independence

The system instructions live in `CLAUDE.md` (Claude Desktop's convention). The content is not Claude-specific. Any AI that can read markdown instructions and read/write local files can operate this system. The README documents how to adapt it to other platforms.

### Rejected alternatives

**Databases (SQLite, PostgreSQL, etc.):** Databases require specific software to read. A database file from 2026 may need specific software versions to open in 2036. JSON files need a text editor.

**Vector stores and embeddings (for knowledge retrieval):** Embedding-based search requires a vector database, an embedding model, and re-indexing when models change. The entity index provides deterministic lookup with no infrastructure. The entity INDEX is a flat JSON file that scales linearly. The AI's context window is the only constraint on how many files the system can manage, and context windows grow with each model generation.

**Cloud storage or sync services:** Third-party dependencies introduce privacy concerns and failure modes the user can't control. The folder is the system. Backup is the user's responsibility, with guidance from the system.

## Designed for data integrity

Health data errors compound. A wrong medication name propagated across files could affect treatment decisions months later. The architecture treats data integrity as a structural property, not a feature to be tested.

### Three layers with strict truth ownership

```
my-data/              reference/               research/
(personal facts)      (interpretations)        (documented evidence)

Source of truth:      Derived from both.       Source of truth:
what happened         Updated only in          what the evidence
to this person        conversation with        shows and how it
                      the user.                was established
```

**my-data/** stores facts about the person: lab results, medications, symptoms, provider visits, decisions. These files are the source of truth for what happened to this person. They are never modified by changes to reference or research files.

**research/** stores documented evidence: study findings, traced mechanisms, treatment profiles, methodology assessments. These files are the source of truth for the evidence base. They are never modified by changes to personal data or reference files.

**reference/** stores interpretations: what the personal facts mean in context of the evidence. This is the only derived layer. It draws from both my-data/ and research/ through pointers, never copies. It is the only layer that gets updated when either source layer changes.

Data flows in one direction: my-data/ and research/ feed into reference/. Never the reverse.

### Pointers, not copies

Reference files never contain copies of raw values from my-data/ or findings from research/. They contain pointers (which file, which entries) and interpretations (what the combination means for this person).

This decision was reached by evaluating three approaches:

**Option A (materialized view):** Reference files contain copies of data. "Ferritin: 18 ng/mL, below optimal." Problem: the value exists in two places. When lab-results.json updates, the reference is stale until someone propagates the change. Between the update and the propagation, the system has contradictory data.

**Option B (computed at read time):** No reference files at all. The AI reads my-data/ and research/ every session and derives interpretations on the fly. Problem: the interpretations are ephemeral. The AI might make a connection in one session and miss it in the next, depending on which files it happened to read. No compounding. No longitudinal narrative.

**Option C (pointers + interpretation):** Reference files contain pointers to source data and evidence, plus the interpretation and a chronological narrative. The AI follows pointers to get current values. The interpretation may be stale, but the facts the AI reads through pointers are always current. The AI can detect staleness by comparing the interpretation against current data in the same read pass.

Option C was chosen. It provides the compounding benefits of pre-computed synthesis (Option A) without the data duplication risk, and the currency of live computation (Option B) without the ephemeral interpretation problem.

### Write verification

Every write to a data file follows a verification protocol: write the file, read it back, confirm the data is correct. Entry counts are checked before and after. If verification fails, the file is restored from git and the user is alerted. This protocol is defined in `.claude/rules/data-routing.md`.

### Git as invisible infrastructure

The system initializes a git repository in the project folder. Every write is a commit. The user never sees git, never runs git commands, never knows it exists. Git provides:

- Complete history of every change ever made
- Restore point before updates
- Rollback capability if a write goes wrong
- Audit trail for debugging

## Designed to compound

The most common failure mode of personal knowledge systems is that they accumulate data without building understanding. Files are created and never revisited. The system knows more but understands the same amount. Health Charted's knowledge layer is designed to get richer over time, not just larger.

### Entity index

Every research and reference file declares the entities it covers in its INDEX.json entry. Entities are canonical names for conditions, medications, lab markers, mechanisms, interactions, symptoms, and providers.

```json
{
  "file": "01-iron-metabolism.md",
  "topic": "Iron absorption, transport, storage mechanisms",
  "entities": {
    "conditions": ["iron-deficiency", "anemia"],
    "lab_markers": ["ferritin", "transferrin-sat", "hemoglobin"],
    "medications": ["ferrous-bisglycinate", "ferrous-sulfate"],
    "mechanisms": ["hepcidin-regulation", "calcium-iron-competition"]
  }
}
```

The entity index enables four operations:
- **Cross-referencing:** find all files that discuss a specific entity
- **Gap detection:** identify active medications or conditions with no research coverage
- **Propagation:** when source data changes, find which reference files to flag
- **Tiered loading:** in large knowledge bases, load only files relevant to the current session's topic

Entity names follow the naming discipline defined in `.claude/rules/data-routing.md`. The same canonical name for a medication appears in my-data/treatments.json, in research INDEX entities, and in reference INDEX entities. That consistency is what makes the join work.

### Propagation as flagging

When my-data/ changes (new lab result, new medication, new symptom), the AI uses the entity index to find reference files that cover the affected entities. It does not rewrite those files. It flags them in status.json and surfaces the flag to the user: "Your new lab results may affect the assessment in your iron management reference. Want me to update it?"

If the user approves, the AI reads the reference file, follows the pointers to get current data, updates the interpretation and narrative, and removes the flag. If the user defers, the flag persists and surfaces again next session.

The same process applies when research/ changes: new or updated research triggers flags on reference files that share entities.

Every interpretation update happens in conversation with the user.

### Gap detection

At session start, the AI compares entities in my-data/ (active conditions, current medications) against entity coverage in both INDEX files. Gaps are surfaced: "You're taking levothyroxine for hypothyroidism, but we don't have any research on file for either. Want me to look into it?"

This is the "system knows what it doesn't know" property. Without the entity index, gaps are invisible. The AI would have to read every research file and every medication to notice that a treatment has no research coverage.

### Reference file format

Reference files use a pointer+interpretation format:

```markdown
## Iron status

**Sources:**
- Data: my-data/lab-results.json (ferritin entries)
- Evidence: research/01-iron-metabolism.md (Optimal Ranges)

**Interpretation:**
Below functional optimal. Improving from baseline.
Current supplementation protocol producing gradual improvement.

**Narrative:**
- 2026-03: diagnosed, started supplementation
- 2026-06: improving, continue protocol
- 2026-09: strong improvement, approaching target

**Next action:**
Recheck Dec 2026. Target: 50+.

**Cross-references:**
- Calcium timing: see supplement-protocol.md
```

The Sources section contains pointers. The Interpretation section contains the only original content. The Narrative section is append-only (new entries added, old entries never modified). Over years, the narrative becomes the person's health biography for that topic.

### Rejected alternatives

**Calendar-based staleness (review_by):** Earlier versions included a `review_by` date on research files. The AI would flag files past their review date. This was removed because you can't know whether research is stale without doing new research to find out. A 5-year-old study on basic biochemistry may still be current. A 3-month-old study on an emerging treatment may already be superseded. The review_by date was a guess masquerading as a check. Staleness is now event-driven: research is flagged for review when new data or evidence arrives for the same entities, not when a calendar date passes.

**Auto-rewriting propagation:** Instead of flagging reference files for user review, the system could automatically rewrite them when source data changes. This was rejected because the integrity risk compounds over time. Each auto-rewrite is an opportunity for the AI to subtly alter existing content: dropping a qualifier, rewording a nuance, conflating similar findings. Over hundreds of propagation cycles, the reference drifts from the source material. Flagging keeps the user in the loop at the interpretation layer, where errors are most consequential.

**Entity pages (dedicated files per entity):** Instead of entity entries in INDEX.json, the system could maintain dedicated files for each entity (one for metformin, one for ferritin, etc.). This was rejected because it duplicates content from research and reference files and creates a maintenance burden. The entity index gives entity-level access (find all files about metformin) without entity-level files. If a user's needs eventually outgrow this, entity pages can be added as a reference file type without architectural changes.

**Automatic answer capture:** When the AI researches a question during conversation, the answer could be automatically filed as a new reference or research file. This was rejected because it generates noise. Most ad-hoc answers don't warrant their own file. Over time, accumulated answer files create maintenance debt and dilute the proactive surfacing system with low-value review flags. Instead, the AI can offer to persist a valuable synthesis when it's warranted. Selective and conversational, not automatic.

**Lint as a separate operation:** A dedicated "lint" mode could check the knowledge base for contradictions, orphan files, and inconsistencies. This was rejected because the checks that matter (stale interpretations, missing coverage, pending flags) already run during proactive surfacing at session start. A separate mode adds process without adding capability.

**derived_from field on reference INDEX entries:** Reference INDEX entries could include a `derived_from` field listing which my-data/ and research/ files they draw from. This was removed because it's redundant with entity-based lookup. When a research file changes, the entity overlap between the research file and reference files identifies the same affected files that derived_from would. Provenance (where a reference came from) is already recorded in the reference file body's Sources section, which travels with the file.

**last_interpretation_update field on reference INDEX entries:** Reference INDEX entries could include a date recording when the interpretation was last verified. This was removed because it's redundant with the propagation flagging system. If pending_reference_updates has a flag for a file, the interpretation needs review. If there's no flag, no source data has changed since the last update. The flag system is event-driven. The date field would be a timestamp of the last event, adding no information the flag doesn't already provide.

## Designed to be extensible

The system is built to grow without breaking. New capabilities, new data types, and new integrations should be additive, never requiring changes to existing data or behavior.

### Entity taxonomy

The entity index uses an open-ended taxonomy. Entity types are string keys in a JSON object. Current types: conditions, lab_markers, medications, mechanisms, interactions, symptoms, providers. Future types (genomic_markers, imaging_markers, wearable_data) are added as new keys. No schema migration required. Existing files continue to work without modification.

### MANIFEST-based updates

The MANIFEST file lists every system file in the repo. During updates, only listed files are replaced. Everything else is personal data. New system files added in future versions get picked up automatically through the remote MANIFEST.

### Guides as behavioral configuration

The system's behavior is defined in markdown guides, not in code. The AI reads these guides at session start and follows them. Changing the system's behavior means editing a guide. Adding a new capability means adding a new guide and referencing it from CLAUDE.md. This makes the system modifiable by anyone who can write clear instructions in plain language.

### Research engine as a plugin

Deep research capabilities come from the research-engine plugin, installed separately. This keeps the core system dependency-free while allowing powerful research for users who set it up. The plugin architecture means other research tools can be integrated the same way without modifying the core system.

## Designed for the user

The person using this system may not be technical. Every design choice that affects the user experience optimizes for clarity over capability.

### No technical language in conversation

The AI never shows file paths, JSON structures, git commands, or error messages in conversation unless the user asks. A translation table in CLAUDE.md maps internal concepts to natural language: "your health profile" not "health-profile.json," "your lab results" not "lab-results.json."

### Invisible infrastructure

Git versioning, write verification, entity indexing, propagation flagging, and backup all happen without the user's awareness. The user talks about their health. The system handles everything else.

### Updates as a conversation

When a new version is available, the user sees: "An update is available. [What changed in plain language]. Want me to update?" One question, one answer. The system backs up data, downloads the update, and verifies it. If anything goes wrong, it restores automatically. The user never sees git, GitHub, manifests, or file operations.

### Onboarding as a first visit

The first session is modeled on a first visit with a new doctor: what brings you here, health background, medications, lifestyle. One question at a time. Conversational. Persistent across sessions (if the user leaves mid-intake, it picks up where they left off next time).

## File inventory

### System files (replaced during updates)

| File | Purpose |
|------|---------|
| `CLAUDE.md` | System instructions: identity, behavior, session initialization, rules |
| `ARCHITECTURE.md` | This document |
| `README.md` | User-facing documentation |
| `CHANGELOG.md` | Version history |
| `VERSION` | Current version number and date |
| `MANIFEST` | List of system files (the update allowlist) |
| `LICENSE` | CC BY-NC-SA 4.0 |
| `.claude/settings.json` | Permission auto-approval for health data operations |
| `guides/*.md` | Behavioral guides (11 files covering protocol, data routing, research, onboarding, updates, etc.) |
| `schemas/*.json` | JSON Schema definitions for all data file types (11 files) |

### Personal data (never touched by updates)

| File | Purpose |
|------|---------|
| `my-data/health-profile.json` | Identity, conditions, goals, providers, medications, family history |
| `my-data/lab-results.json` | Timestamped lab results with context and interpretation |
| `my-data/treatments.json` | Medications, supplements, therapies, environmental modifications |
| `my-data/symptoms.json` | Observations with severity, triggers, pattern analysis |
| `my-data/lifestyle.json` | Sleep, diet, stress, exercise, lifestyle factors |
| `my-data/visit-notes.json` | Provider visit summaries, orders, follow-ups |
| `my-data/decisions.json` | Decision log with reasoning (never pruned) |
| `my-data/status.json` | Current priorities, pending items, onboarding state, propagation flags |
| `preferences.json` | User overrides for system defaults |
| `reference/INDEX.json` | Index of reference files with entity coverage |
| `reference/*.md` | User's reference files (pointer+interpretation format) |
| `research/INDEX.json` | Index of research files with entity coverage |
| `research/*.md` | User's research files (deep, cited evidence) |
