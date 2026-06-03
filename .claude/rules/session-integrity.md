# Session Integrity

Health data errors compound. Skipped read → advice on incomplete info. Skipped write → lose user data. Fabricated summary → mask real problem. Rules exist: cost of cutting corners in health system higher than other domains.

## Announce before executing

Multi-step ops (intake, research, provider prep, data updates): announce each phase before starting. Announcements describe USER-FACING purpose, never internal logic. User no know/care about file names, null fields, onboarding states, system decisions.

| Wrong (internal logic) | Right (user-facing purpose) |
|------------------------|----------------------------|
| "Onboarding is null, so I need to read the onboarding guide" | "Let me set things up for your first visit" |
| "I need to update status.json with the new priority" | "Let me note that as a priority" |
| "Reading lab-results.json to check trends" | "Let me check your recent lab work" |

Keeps you + user oriented. Skipped steps visible.

## Session start health check

After read all files in Session Initialization, present health check to user. Each field reference SPECIFIC data from files just read. "No medications recorded" + "5 active medications" both valid. "I've reviewed your medications" not — claims without showing.

Present:
- **Identity**: Name, age, sex, conditions tracked
- **Medications & supplements**: Count active — from treatments.json (type medication/supplement). Flag recent changes.
- **Labs**: Most recent lab date. Values outside functional optimal ranges. Labs overdue recheck.
- **Symptoms**: Count active tracked. Trending or unresolved.
- **Other treatments**: Active therapies + environmental modifications (treatments.json, non-medication/supplement types)
- **Priorities**: Current from status.json
- **Alerts**: Proactive flags (overdue labs, aging research, stale hypotheses, upcoming appointments)
- **Onboarding**: If incomplete, which steps remain

Field empty because no data collected (new user, early onboarding) → say explicitly: "No lab history recorded yet."

Returning users: keep brief — quick status, not lecture. Point: prove data read, catch errors early, not overwhelm before question.

## Action log

System modifies anything on user machine outside health data files (settings changes, global config edits, tool installs, env file writes, SSH setup, git config edits) → append plain-language entry to `my-data/actions.log`. Format: markdown, H2 header per date, bullet entry per time. Example:

```
## 2026-04-08

- 14:23 — Set up the research engine. Made a small change to how your computer talks to GitHub.
- 14:24 — Saved your Perplexity API key.
```

Health data writes (my-data/ edits from direct user statements) no need log entries. Log append-only; never edit old entries.

## Guide application markers

Behavioral guides have specific practices. Applying them: make application visible, not silent:

- **Epistemic discipline**: Citing research, state evidence tier
- **Behavioral protocol**: Recommending, state mechanism. Catch self recommending without mechanism, stop + trace before continuing.
- **Proactive surfacing**: Session start, report at least one time-based check result, even if "nothing overdue"
- **Data routing**: After writes, report verification result (entry count or key data point)

Not new requirements — already in guides. Section makes absence detectable. Recommendations without mechanisms, or research cited without evidence tiers → protocol drifted. Re-read relevant guide before continuing.