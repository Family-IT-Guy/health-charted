# Data Routing

How to capture, route, and cross-reference information throughout every conversation.

## Core Rule

Information capture runs continuously, not as a discrete step. When the user shares something that affects their health, capture it immediately in the appropriate file. Do not defer to the end of the session. Do not batch updates.

When capturing, briefly note what was recorded and why it matters. Don't announce every field change verbosely.

## Data Format

All data files are JSON. Schemas are defined in `schemas/`. Each file has a `_type` field that identifies its schema. When writing to a data file, follow the schema for that file type. If a field is unknown, use `null`. If an array has no entries, use `[]`.

When a schema adds new fields in a future version, add them with null values the next time you write to that file. Do not remove or rename existing fields.

## Write Safety

Data accumulated over years is irreplaceable. Every write must be verified.

1. **Before writing**: count the entries in each array in the file.
2. **Write the file.**
3. **After writing**: read the file back. Verify:
   - The JSON is valid
   - All entries that existed before the write still exist
   - Entry counts match expected (previous count + additions - removals)
   - No fields were dropped from any entry
   - New data was written correctly
4. **If verification fails**: restore the file from the last git commit (`git checkout HEAD -- <file>`), alert the user, and do not retry the write without understanding what went wrong.

## Naming Discipline

Medications, supplements, conditions, and providers each have a canonical name that must be used consistently across all data files.

**Canonical name sources:**
- Medications and supplements: the `name` field in `my-data/treatments.json`
- Conditions: the `condition` field in `my-data/health-profile.json`
- Providers: the `name` field in the providers array in `my-data/health-profile.json`

When referencing any of these entities in any data file, use the exact canonical name. Do not abbreviate, do not include dosage in the name, do not use alternate spellings.

When a new name appears that is similar to an existing canonical name (abbreviation, alternate spelling, brand vs generic, provider terminology), ask the user to confirm whether they are the same entity before merging. Never assume.

When a provider uses different terminology for a condition or treatment, record the provider's term in the `provider_terminology` field of the visit entry, but use the canonical name for cross-references.

## Severity Scale

Default symptom severity scale (1-5):

1. Minimal: barely noticeable, no functional impact
2. Mild: noticeable but not disruptive
3. Moderate: clearly present, some functional impact
4. Significant: frequent or intense, interfering with activities
5. Severe: near-constant or very intense, significant distress or impairment

The user can override this scale in `preferences.json` under `severity_scale`. If an override exists, use it instead of the default.

## Filter

Before capturing, ask: "Does this information affect the user's health management, treatment decisions, or interpretation of their data?"

If yes, capture it. If no, don't.

"Had a busy weekend" — don't capture. "Didn't sleep well all week because of work stress" — capture (sleep disruption affects multiple health pathways).

## Routing Table

| Information type | Examples | Target file | Cross-reference |
|-----------------|----------|-------------|-----------------|
| Lab results | Blood work values, test results, imaging findings | `my-data/lab-results.json` | Check against treatments.json for medication effects. Check reference files for optimal ranges. Compare to previous values for trends. |
| Symptom observations | New symptoms, severity changes, pattern changes, triggers identified | `my-data/symptoms.json` | Correlate with recent entries in lifestyle.json and treatments.json. Check for temporal relationships. |
| Treatment changes | Medication started/stopped/adjusted, supplement changes, therapy sessions, lifestyle interventions | `my-data/treatments.json` | Check for interactions with existing treatments. Update monitoring schedule. Note in status.json if follow-up needed. |
| Environment changes | Sleep patterns, diet changes, exercise, stress, living situation, work, relationships | `my-data/lifestyle.json` | Correlate with symptom patterns. Check for nutritional or physiological implications. |
| Provider interactions | "Doctor said X", test ordered, appointment scheduled, referral made, recommendation given | `my-data/visit-notes.json` | Route extracted data to all relevant files (lab orders to status.json as pending, medication changes to treatments.json, etc.). Flag conflicts with current understanding. |
| Decisions | Treatment chosen or rejected, strategic health decisions, approach changes | `my-data/decisions.json` | Record reasoning, options considered, and evidence. Prevents re-recommendation in future sessions. Reference from other files when relevant. |
| Action items | New tasks, completed items, upcoming deadlines, scheduled appointments | `my-data/status.json` | Remove when resolved. Note completion in the relevant file. **Always use the full object shape:** `{"item": "...", "date_added": "YYYY-MM-DD", "deadline": null, "status": "pending"}`. Never write action items as bare strings. `date_added` is required for aging detection. |
| Profile changes | New provider, new diagnosis, medication list update, insurance change, family history update | `my-data/health-profile.json` | May affect interpretation of existing data. Flag if a profile change means previous recommendations should be reassessed. |

## Cross-Referencing

When routing data to a file, always check for connections to other files:

- **New lab result** — Does this change the interpretation of current symptoms? Does it validate or invalidate an open hypothesis? Does it affect the treatment stack (dose adjustment, new deficiency)?
- **New symptom** — Does this correlate with a recent environment change? A treatment change? A lab trend? Does it match a pattern from months ago?
- **Treatment change** — What should be monitored? When should the effect be reassessed? Are there interactions with existing treatments?
- **Environment change** — Could this explain recent symptom changes? Does it affect lab interpretation (stress affects cortisol, sleep affects multiple markers)?
- **Provider input** — Does this conflict with the system's current understanding? If so, flag for the user with both sides of the evidence.

These cross-references are where the system adds value beyond simple record-keeping. A human tracking their health in a notebook captures data. This system captures data AND surfaces connections.

## Propagation Flagging

When my-data/ changes, the change may invalidate interpretations in reference files. Use the entity index to identify affected reference files and flag them for review.

### After every my-data/ write:

1. **Identify entities affected.** What condition, medication, lab marker, or symptom was updated?
2. **Search reference/INDEX.json** for files whose `entities` include the affected entity names.
3. **For each affected reference file**, check whether the change is meaningful:
   - New lab result for a tracked marker: meaningful (may change interpretation)
   - Medication dose change: meaningful (may affect monitoring schedule)
   - New symptom entry with same severity as previous: not meaningful (no interpretation change)
4. **For meaningful changes**, add a flag to `my-data/status.json` in `pending_reference_updates`:

   ```json
   {
     "file": "iron-management.md",
     "reason": "New ferritin result (Sep 2026) may change iron status interpretation",
     "triggered_by": "my-data/lab-results.json",
     "date_flagged": "2026-09-15"
   }
   ```

5. **Surface the flag conversationally**: "Your new ferritin result may change the assessment in your iron management reference. Want me to update it now?"
6. **If user approves**: read the reference file, follow pointers to get current data, update the interpretation and narrative sections. Remove the flag from status.json.
7. **If user defers**: the flag persists in status.json and surfaces again at the next session start (via proactive surfacing).

### After research/ changes:

Same process but triggered by research updates instead of my-data/ changes:

1. Identify entities in the new or updated research file.
2. Search reference/INDEX.json for files whose `entities` overlap with the research file's entities.
3. Flag affected reference files in status.json.
4. Surface and update with user approval.

### What propagation is NOT:

- Propagation does not auto-rewrite reference files. Every interpretation update happens in conversation with the user.
- Propagation does not modify my-data/ or research/ files. Data flows down (into reference/), never up.
- Propagation does not create new reference files. It only flags existing ones for review.

## Hypothesis Linkage

When data routing reveals a pattern or raises a question, document it as a hypothesis in the relevant data file:

- State the hypothesis clearly
- Note the evidence that prompted it
- Mark `hypothesis_status` as `open`
- Link to the related data entries across files by referencing canonical names

When subsequent data arrives that validates or invalidates a hypothesis, update it immediately with the new evidence and reasoning. See `guides/epistemic-discipline.md` for full hypothesis tracking rules.
