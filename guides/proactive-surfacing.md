# Proactive Surfacing

At the start of each session, after reading all data files, check for time-sensitive items to surface. Only surface items grounded in specific reasoning. Brief and actionable — a nudge with the reason, not a lecture.

## Triggers

### Overdue Measurements
Compare dates in `my-data/lab-results.json` against recommended recheck intervals in reference files. If time since last measurement exceeds the interval, surface it with the reason the measurement matters.

### Trending Data
Look for directional trends across multiple data points in lab-results.json or symptoms.json. A single data point is not a trend. Three or more values moving in the same direction is. Surface the trend with the rate of change and what it might mean mechanistically.

### Treatment Reassessment
Check `my-data/treatments.json` for treatments with trial periods or reassessment dates. If a trial period is ending or an intervention hasn't been reassessed in a reasonable timeframe, surface it.

### Upcoming Provider Visits
Check `my-data/status.json` and `my-data/visit-notes.json` for scheduled appointments. If one is approaching, offer to prepare: generate questions based on current data, identify gaps to address, suggest labs to request.

### Pending Action Items
Check `my-data/status.json` for unresolved items approaching a deadline or aging without progress. Surface with the original context.

### Cross-Temporal Patterns
When reading the full dataset at session start, watch for correlations that span weeks or months:
- Symptom patterns that align with environment changes
- Lab trends that correlate with treatment changes
- Seasonal or cyclical patterns emerging from accumulated data

Surface these as observations, not conclusions. "I notice X correlates with Y over the past 3 months. Worth investigating?"

### Research Aging
Check research files in `research/` for YAML frontmatter containing `review_by` dates. If a research file's `review_by` date has passed, surface it: "Your research on [topic] was created on [date_created] and is due for review. Medical knowledge in this area may have evolved. Want to check for updates?"

If the user wants to update, run a targeted research refresh per `guides/research-guide.md`. If they defer, update the `review_by` date in the frontmatter to a new date agreed with the user.

### Stale Hypotheses
Check data files for hypotheses marked "open" or "untested" that haven't been revisited. If a hypothesis has been open for an extended period without new data, surface it: "We have an open hypothesis from [date] about [topic]. Should we investigate, or has the situation changed?"

## Format

When surfacing items at session start:

- Lead with the most time-sensitive or health-critical item
- One or two sentences per item: what it is and why it matters now
- Group related items together
- Don't overwhelm. If there are many items, prioritize and ask if the user wants to see the rest
- After surfacing, ask what the user wants to focus on. Don't assume the surfaced items are more important than what they came to discuss
