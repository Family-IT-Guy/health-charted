# Entity Context Verification

Health data accumulated over years is irreplaceable context. Failing to query loaded data before substantive engagement on a topic produces wrong recommendations and contaminates research dispatches with false-absence framings (e.g., recommending "test for X" when X is already tested). This rule prevents that error class by anchoring every entity reference in actual loaded data before any output that names it.

## Core rule

When a health-relevant entity enters the conversation — whether user-introduced or framework-introduced — verify it against loaded data before substantive engagement.

An entity is any named marker, treatment, supplement, medication, condition, symptom, genotype variant, mechanism, or test.

## Proactive trigger (first-introduction check)

Fires the first time an entity is introduced in a session.

1. **Cluster expansion via INDEX.** Grep `research/INDEX.json` and `reference/INDEX.json` for the entity. For each file matched, read its `entities` array. The union is the verification cluster.

2. **Verify the cluster against my-data/.** For each cluster member, grep the appropriate my-data file:
   - Markers → `lab-results.json`
   - Treatments / supplements / medications → `treatments.json`
   - Conditions, providers, family history → `health-profile.json`
   - Symptoms → `symptoms.json`
   - Lifestyle factors → `lifestyle.json`
   - Decisions referencing the entity → `decisions.json`
   - Visit-level mentions → `visit-notes.json`

   Collect loaded values, dates, and statuses.

3. **Surface as lightweight inline.** Before any substantive recommendation, research dispatch, or framework synthesis on the entity, present:

   ```
   Records check: [marker] = [value] ([date]); [treatment] = [status]; [condition] = [diagnosed_date]. No data on: [list]. Proceeding.
   ```

   Brief, factual, no narration. The explicit "no data" list is the absence-claim verifier — if it contains an item that the framework is about to recommend testing, the contradiction is visible before the recommendation is written.

## Defensive backstop (entity-tracking)

Maintain a session-level mental list of entities that have been proactively checked.

Before any output that names an entity, confirm the entity is on the checked-list. If not, run the proactive check now and revise the draft output before sending.

This catches the case where a new entity enters via the framework's own output mid-turn (the failure mode where the framework introduces an entity in its own recommendation without the proactive check having fired).

The backstop operates on the agent's own draft output before it is sent. It is not phrase-matching on absence claims ("uncharacterized," "no data on") — it is entity-name detection on whatever the draft contains. Imperfect by nature (the same attentional mechanism that produced the original miss is also what scans the draft), but cheap and the second look catches a real fraction of cases the first pass missed.

## Edge cases

- **Entity not in any INDEX:** apply the rule anyway. Grep my-data/ literally for the term. If nothing returns, surface "No prior coverage on [entity]." Then proceed; do not block.
- **Entity in INDEX but my-data has no related entries:** surface as an explicit gap. Honest absence is the goal — not synthesized completeness.
- **Already-loaded data with stale dates:** surface the date alongside the value. The user evaluates staleness; the framework does not silently treat old data as current.

## Relationship to existing rules

- `proactive-surfacing.md` runs session-start coverage checks (entity gaps, overdue measurements, stale hypotheses). This rule runs real-time entity-introduction checks. Complementary timing.
- `behavioral-protocol.md` Step 4 (Contextualize) extends to every entity introduction under this rule, not only after research returns.
- `epistemic-discipline.md` "Query Before Claiming" is the principle this rule operationalizes for health-data claims.
- `data-routing.md` defines which my-data file holds which entity type — this rule consumes that routing.
