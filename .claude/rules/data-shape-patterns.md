# Data Shape Patterns

How to shape data when capturing it. These patterns prevent classes of structural error that would otherwise compound across years of accumulated data. Apply them whenever writing to any `my-data/` file, and use them as the default shape for any new fields the schema does not yet declare.

These are positive patterns, not validation rules. Schema declares canonical shape; these patterns guide shape decisions when the schema does not yet cover the case.

## Time-varying state

When the same underlying value is measured or recorded multiple times, store the history as an array of dated objects. Never embed the date in a field name.

**Pattern:**

```json
"verifications": [
  { "date": "2026-05-03", "source": "label", "values": { ... } },
  { "date": "2026-09-01", "source": "label", "values": { ... } }
]
```

**Anti-pattern:**

```json
"verified_2026-05-03": { ... },
"verified_2026-09-01": { ... }
```

Why: dynamic-date keys cannot be declared in a JSON schema, cannot be queried by date range, and proliferate one-per-snapshot forever. Array-of-objects is the same shape `lab-results.json` uses for `entries[].date` and that any time-series data should follow.

This applies to verification logs, dose revisions, functional-state snapshots, status transitions, and any other data where the field captures a state at a point in time.

## Lifecycle states

When a tracked entity (treatment, condition, plan, decision, anything with phases) has a current state plus a transition history, capture three fields:

- `<entity>_status` — enum value naming the current state
- `<entity>_status_since` — ISO date the current state began
- `<entity>_status_notes` — free text for rationale or context

**Pattern (treatments.json):**

```json
{
  "name": "Carlson fish oil",
  "lifecycle_status": "active",
  "lifecycle_status_since": "2026-05-03",
  "lifecycle_status_notes": "Increased from 1 tsp to 1.5 tsp for DFI window."
}
```

**Anti-pattern:**

```json
"status": "available_unused → planned_active 2026-05-03"
```

Why: free-text encodings of state transitions vary across sessions ("planned active 2026-05-03", "active since May", "now active"), defeating any query of "what's currently active?" The triple separates the three pieces of information the encoding tries to combine.

Transition history lives in git log (the file itself was different yesterday) and in `decisions.json` (when the transition was a decision). Do not duplicate transition history into the data file.

Each schema declares its own enum values. Treatments, conditions, plans, and decisions each have lifecycle states specific to that entity type; the shape (status_enum, status_since, status_notes) is what generalizes, not the enum values.

## Schema as canonical shape, not contract

When writing a field the schema does not declare, the schema is the canonical-shape reference, not a hard validator. If a sensible field already exists in the schema for what you are capturing, use it. If not, extend the file with a new field that follows these shape patterns. Never rename or remove existing fields (per ARCHITECTURE.md additive-only rule). The next maintainer review will fold the new field into the schema if it generalizes.

The two anti-patterns above (dynamic-date keys, free-text lifecycle transitions) are the main shape errors that have compounded across this system. Avoid them by default. Other shape decisions are made at write-time using judgment.
