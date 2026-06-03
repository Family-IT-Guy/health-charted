# Data Routing

How capture, route, cross-reference info every conversation.

## Core Rule

Info capture runs continuously, no discrete step. User shares something affects health → capture immediately appropriate file. No defer session-end. No batch updates.

When capturing, briefly note what recorded + why matters. No verbose announce every field change.

## Data Format

All data files JSON. Schemas defined `schemas/`. Each file has `_type` field identifies schema. Writing to data file → follow schema for that file type. Field unknown → use `null`. Array no entries → use `[]`.

Schema adds new fields future version → add with null values next write. No remove or rename existing fields.

## Write Safety

Data accumulated over years irreplaceable. Every write must verify.

1. **Before writing**: count entries each array in file.
2. **Write the file.**
3. **After writing**: read file back. Verify:
   - JSON valid
   - All entries existed before write still exist
   - Entry counts match expected (previous count + additions - removals)
   - No fields dropped from any entry
   - New data written correctly
4. **Verification fails**: restore file from last git commit (`git checkout HEAD -- <file>`), alert user, no retry write without understanding what went wrong.

## Write Confirmation

Read-back IS confirmation. No confirm write based on what intended write or remember writing. Confirm based on what read back from file.

After verified write, confirm to user natural language: what changed + what file now contains. Example: "I've added your mother's health history to your records: hypertension (onset age 52) and type 2 diabetes (onset age 60). You now have 4 family members documented."

Haven't written yet → don't confirm. User never hear "I've noted that" or "I've saved that" before write happened + verified.

## Naming Discipline

Named entities join across files by a stable `slug`, never by display name. Each entity has an immutable kebab-case `slug` assigned once at creation (e.g. `metformin`, `iron-deficiency`) plus a mutable `name` (or `condition`) display label and optional `aliases[]`. The slug is what every cross-file reference points at; it never changes.

**Slug homes (where each entity is defined):**
- Medications, supplements, therapies, environmental modifications: `slug` in `my-data/treatments.json`
- Conditions: `slug` in `my-data/health-profile.json` (conditions array)
- Providers: `slug` in `my-data/health-profile.json` (providers array)
- Genomic variants: `slug` in `my-data/genomic-markers.json`
- Lab markers: the marker slug used in `my-data/lab-results.json`; the marker's display name + aliases live in the entity index

**Assigning a slug:** kebab-case the entity's short canonical name. Unique within its type. A genuine collision (two different providers → `john-smith`) gets a meaningful qualifier (`john-smith-cardiology`), never a numeric suffix.

**Referencing:** point at the slug from any file — e.g. a treatment's `treats` lists the slugs of the conditions it treats.

**Renames + merges (never touch the slug):** a name evolves (brand→generic, reclassification) → change the `name` label and/or add the old name to `aliases[]`; the slug stays. Two names turn out to be the same entity → add one to the other's `aliases[]`; never rewrite references. A new entity looks similar to an existing one → ask the user to confirm before merging. Never assume.

Provider uses different terminology for a condition or treatment → record the provider's term in the `provider_terminology` field of the visit entry, but reference the entity by its slug.

## Severity Scale

Default symptom severity scale (1-5):

1. Minimal: barely noticeable, no functional impact
2. Mild: noticeable but not disruptive
3. Moderate: clearly present, some functional impact
4. Significant: frequent or intense, interfering with activities
5. Severe: near-constant or very intense, significant distress or impairment

User can override scale in `preferences.json` under `severity_scale`. Override exists → use instead of default.

## Filter

Before capturing, ask: "Does this information affect the user's health management, treatment decisions, or interpretation of their data?"

Yes → capture. No → don't.

"Had a busy weekend" → don't capture. "Didn't sleep well all week because of work stress" → capture (sleep disruption affects multiple health pathways).

## Routing Table

| Information type | Examples | Target file | Cross-reference |
|-----------------|----------|-------------|-----------------|
| Lab results | Blood work values, test results, imaging findings | `my-data/lab-results.json` | Check against treatments.json for medication effects. Check reference files for optimal ranges. Compare to previous values for trends. |
| Symptom observations | New symptoms, severity changes, pattern changes, triggers identified | `my-data/symptoms.json` | Correlate with recent entries in lifestyle.json + treatments.json. Check for temporal relationships. |
| Treatment changes | Medication started/stopped/adjusted, supplement changes, therapy sessions, lifestyle interventions | `my-data/treatments.json` | Check interactions with existing treatments. Update monitoring schedule. Note in status.json if follow-up needed. |
| Environment changes | Sleep patterns, diet changes, exercise, stress, living situation, work, relationships | `my-data/lifestyle.json` | Correlate with symptom patterns. Check for nutritional or physiological implications. |
| Provider interactions | "Doctor said X", test ordered, appointment scheduled, referral made, recommendation given | `my-data/visit-notes.json` | Route extracted data to all relevant files (lab orders to status.json as pending, medication changes to treatments.json, etc.). Flag conflicts with current understanding. |
| Decisions | Treatment chosen or rejected, strategic health decisions, approach changes | `my-data/decisions.json` | Record reasoning, options considered, evidence. Prevents re-recommendation future sessions. Reference from other files when relevant. |
| Action items | New tasks, completed items, upcoming deadlines, scheduled appointments | `my-data/status.json` | Remove when resolved. Note completion in relevant file. **Always use full object shape:** `{"item": "...", "date_added": "YYYY-MM-DD", "deadline": null, "status": "pending"}`. Never write action items as bare strings. `date_added` required for aging detection. |
| Profile changes | New provider, new diagnosis (condition), allergy, insurance change | `my-data/health-profile.json` | Conditions and providers each get a `slug`. May affect interpretation of existing data. Flag if a profile change means previous recommendations should be reassessed. NOT medications/supplements (→ treatments.json) and NOT family history (→ family-history.json). |
| Family history | A relative's conditions, age of onset, cause of death — hereditary-risk context | `my-data/family-history.json` | One entry per relative. Reference each condition by its slug (same namespace as health-profile.conditions). Drives screening/risk reasoning. |
| Genomic findings | Genetic variants, methylation states, pharmacogenomic phenotypes | `my-data/genomic-markers.json` | Variants get a `slug`. Cross-reference to treatments (metabolism) and lab interpretation. |
| Photos | Progress photos (self over time), aspirational references ("someone I want to look like"), injury/skin/hair documentation | `my-data/photos.json` | Register with unique ID, date, kind, path (relative to user folder), optional angle/conditions/notes. Link photo ID from relevant entry: strategic-plans baseline.photo_ids (progress) or target.aspirational_photo_ids (aspirational), symptoms.photo_ids, visit-notes.photo_ids. Photo files stay local; framework never transmits image data. |
| Training | Structured resistance/performance programming, phase transitions, key-lift progression, injuries affecting training, HRV + recovery tracking | `my-data/training.json` | current_program for active program; append new phases to phase_history; append key-lift progression to history arrays; injuries_and_limitations for ongoing adaptations. Generic activity (walking, recreational sports) stays in lifestyle.json.exercise. Cross-reference injuries to symptoms.json via related_symptom_id where applicable. |
| Strategic plans | New plan creation, plan status changes, reassessment outcomes | `my-data/strategic-plans.json` | Link to related outcome, relevant markers (by slug) from lab-results.json. Link photo_ids from my-data/photos.json if relevant. |

**Intervention vs baseline (the lifestyle discriminator):** `treatments.json` is the single home for every *intervention* — anything deliberately adopted and tracked for effect, with a lifecycle (a medication, supplement, therapy, or a lifestyle change being run as an intervention like a fasting protocol or a training block). `lifestyle.json` holds *baseline context* — what the person's sleep, diet, stress, and general activity are like. The same topic (exercise) can appear as baseline context in lifestyle.json AND as a specific tracked protocol in treatments.json; those are different facts, not a double-home. Discriminator: tracked-for-effect with a lifecycle → treatments; background context → lifestyle.

## Cross-Referencing

Routing data to file → always check connections to other files:

- **New lab result** — Does change interpretation of current symptoms? Validate or invalidate open hypothesis? Affect treatment stack (dose adjustment, new deficiency)?
- **New symptom** — Correlate with recent environment change? Treatment change? Lab trend? Match pattern from months ago?
- **Treatment change** — What should monitor? When effect reassessed? Interactions with existing treatments?
- **Environment change** — Could explain recent symptom changes? Affect lab interpretation (stress affects cortisol, sleep affects multiple markers)?
- **Provider input** — Conflict with system's current understanding? If so, flag for user with both sides of evidence.

Cross-references where system adds value beyond simple record-keeping. Human tracking health in notebook captures data. This system captures data AND surfaces connections.

## Propagation Flagging

my-data/ changes → change may invalidate interpretations in reference files. Use entity index identify affected reference files + flag for review.

### After every my-data/ write:

1. **Identify entities affected.** What condition, medication, lab marker, or symptom updated?
2. **Search reference/INDEX.json** for files whose `entities` include affected entity names.
3. **For each affected reference file**, check whether change meaningful:
   - New lab result for tracked marker: meaningful (may change interpretation)
   - Medication dose change: meaningful (may affect monitoring schedule)
   - New symptom entry same severity as previous: not meaningful (no interpretation change)
4. **For meaningful changes**, add flag to `my-data/status.json` in `pending_reference_updates`:

   ```json
   {
     "file": "iron-management.md",
     "reason": "New ferritin result (Sep 2026) may change iron status interpretation",
     "triggered_by": "my-data/lab-results.json",
     "date_flagged": "2026-09-15"
   }
   ```

5. **Surface flag conversationally**: "Your new ferritin result may change the assessment in your iron management reference. Want me to update it now?"
6. **User approves**: read reference file, follow pointers to get current data, update interpretation + narrative sections. Remove flag from status.json.
7. **User defers**: flag persists in status.json + surfaces again next session start (via proactive surfacing).

### After research/ changes:

Same process but triggered by research updates instead of my-data/ changes:

1. Identify entities in new or updated research file.
2. Search reference/INDEX.json for files whose `entities` overlap with research file's entities.
3. Flag affected reference files in status.json.
4. Surface + update with user approval.

### What propagation is NOT:

- Propagation no auto-rewrite reference files. Every interpretation update happens in conversation with user.
- Propagation no modify my-data/ or research/ files. Data flows down (into reference/), never up.
- Propagation no create new reference files. Only flags existing ones for review.

## Hypothesis Linkage

Data routing reveals pattern or raises question → document as hypothesis in relevant data file:

- State hypothesis clearly
- Note evidence prompted it
- Mark `hypothesis_status` as `open`
- Link to related data entries across files by referencing canonical names

Subsequent data arrives validates or invalidates hypothesis → update immediately with new evidence + reasoning. See Epistemic Discipline rules for full hypothesis tracking rules.