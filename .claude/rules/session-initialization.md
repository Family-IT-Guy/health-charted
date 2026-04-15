# Session Initialization

At the start of every session, read in this order. **Read every file in its entirety. Do not skip lines, skim, or summarize during reading. Health data requires complete context. A skipped line could be a critical lab result, an active medication, or an unresolved hypothesis.**

0. Run `date '+%Y-%m-%d %H:%M %Z'` to establish the current date and time. Health timestamps must be unambiguous. Do not rely on training data or inference for the current date.
1. `my-data/status.json` — current priorities, quick context, pending items, onboarding state
2. `my-data/health-profile.json` — who the user is, their situation, their goals
3. `preferences.json` — user preferences and overrides (if it exists and has content)
4. `reference/INDEX.json` — available reference files
5. Remaining my-data/ files: `lab-results.json`, `treatments.json`, `symptoms.json`, `lifestyle.json`, `visit-notes.json`, `decisions.json`

Behavioral rules (`.claude/rules/`) are auto-loaded. After reading data files, run the proactive surfacing checks.

Check for updates: read the local `VERSION` file, then try to fetch `https://raw.githubusercontent.com/Family-IT-Guy/health-charted/main/VERSION`. If the remote version is newer, follow `guides/update.md` to offer and perform the update. If the fetch fails, skip silently.

## Onboarding Check

After reading `my-data/status.json`, check the `onboarding` field:

- **If null or `status` is `in_progress`:** read `guides/onboarding.md` and follow it.
- **If `status` is `completed`:** skip onboarding, proceed normally.
