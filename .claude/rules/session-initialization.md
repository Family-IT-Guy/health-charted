# Session Initialization

At the start of every session, read in this order. **Read every file in its entirety. Do not skip lines, skim, or summarize during reading. Health data requires complete context. A skipped line could be a critical lab result, an active medication, or an unresolved hypothesis.**

0. Run `date '+%Y-%m-%d %H:%M %Z'` to establish the current date and time. Health timestamps must be unambiguous. Do not rely on training data or inference for the current date.
1. `my-data/status.json` — current priorities, quick context, pending items, onboarding state
2. `my-data/health-profile.json` — who the user is, their situation, their goals
3. `preferences.json` — user preferences and overrides (if it exists and has content)
4. `reference/INDEX.json` — available reference files
5. Remaining my-data/ files: `lab-results.json`, `treatments.json`, `symptoms.json`, `lifestyle.json`, `visit-notes.json`, `decisions.json`, `training.json`, `photos.json`, `strategic-plans.json`
6. **Scan installed modules.** For each subdirectory of `modules/` (if the directory exists):
   - Read `modules/<name>/MODULE.json`
   - Validate `requires_hc_min_version` ≤ current HC core VERSION via semver comparison; if incompatible, surface the incompatibility plainly to the user (e.g., "the [module] module requires HC v[X]; you have HC v[Y]. Want to update HC first?") and skip loading that module's discovery layer
   - Build the discovery layer in working context: module name, namespace, description, top-level domain triggers, top-level salience hints, declared guides, and the recursive submodule tree (each submodule's namespace, version, triggers, hints, intake_questions, covers_entities)
   - Do NOT load module wiki, research, reference, or guide content at this stage. Module content loads on domain trigger.

Behavioral rules (`.claude/rules/`) are auto-loaded. After reading data files, run the proactive surfacing checks.

Check for updates: read the local `VERSION` file, then try to fetch the remote VERSION from this installation's source repo (see `guides/update.md` for the configured update source URL). If the remote version is newer, follow `guides/update.md` to offer and perform the update. If the fetch fails, skip silently.

## Onboarding Check

After reading `my-data/status.json`, check the `onboarding` field:

- **If null or `status` is `in_progress`:** read `guides/onboarding.md` and follow it.
- **If `status` is `completed`:** skip onboarding, proceed normally.
