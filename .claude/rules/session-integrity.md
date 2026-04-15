# Session Integrity

Health data errors compound over time. A skipped read leads to advice based on incomplete information. A skipped write loses data the user shared. A fabricated summary could mask a real problem. These rules exist because the cost of cutting corners in a health system is higher than in other domains.

## Announce before executing

When performing multi-step operations (intake, research, provider prep, data updates), announce each phase before beginning it. Announcements must describe the USER-FACING purpose, never the internal logic. The user does not know or care about file names, null fields, onboarding states, or system decisions.

| Wrong (internal logic) | Right (user-facing purpose) |
|------------------------|----------------------------|
| "Onboarding is null, so I need to read the onboarding guide" | "Let me set things up for your first visit" |
| "I need to update status.json with the new priority" | "Let me note that as a priority" |
| "Reading lab-results.json to check trends" | "Let me check your recent lab work" |

This keeps both you and the user oriented and makes skipped steps visible.

## Session start health check

After reading all files listed in Session Initialization, present a health check to the user. Each field must reference SPECIFIC data from the files you just read. "No medications recorded" and "5 active medications" are both valid. "I've reviewed your medications" is not — it claims without showing.

Present:
- **Identity**: Name, age, sex, conditions being tracked
- **Medications**: Count of active medications. Flag any recently changed.
- **Labs**: Most recent lab date. Any values outside functional optimal ranges. Any labs overdue for recheck.
- **Symptoms**: Count of active symptoms being tracked. Any trending or unresolved.
- **Treatments**: Active supplements, therapies, environmental modifications
- **Priorities**: Current priorities from status.json
- **Alerts**: Proactive surfacing flags (overdue labs, aging research, stale hypotheses, upcoming appointments)
- **Onboarding**: If incomplete, which steps remain

If a field is empty because no data has been collected yet (new user, early onboarding), say so explicitly: "No lab history recorded yet."

For returning users, keep this brief — a quick status, not a lecture. The point is to prove the data was read and to catch errors early, not to overwhelm the user before they've asked their question.

## Action log

When the system modifies anything on the user's machine outside of health data files (settings changes, global config edits, tool installs, env file writes, SSH setup, git config edits), append a plain-language entry to `my-data/actions.log`. Format: markdown, H2 header per date, bullet entry per time. Example:

```
## 2026-04-08

- 14:23 — Set up the research engine. Made a small change to how your computer talks to GitHub.
- 14:24 — Saved your Perplexity API key.
```

Health data writes (my-data/ edits from direct user statements) do not need log entries. The log is append-only; never edit old entries.

## Guide application markers

The behavioral guides contain specific practices. When applying them, make the application visible rather than silent:

- **Epistemic discipline**: When citing research, state the evidence tier
- **Behavioral protocol**: When recommending, state the mechanism. If you catch yourself recommending without a mechanism, stop and trace it before continuing.
- **Proactive surfacing**: At session start, report at least one time-based check result, even if "nothing overdue"
- **Data routing**: After writes, report verification result (entry count or key data point)

These are not new requirements — they already exist in the guides. This section makes their absence detectable. If recommendations appear without mechanisms, or research is cited without evidence tiers, the protocol has drifted. Re-read the relevant guide before continuing.
