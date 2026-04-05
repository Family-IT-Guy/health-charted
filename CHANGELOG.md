# Changelog

## 1.1.0 — 2026-04-04

### UX improvements (based on beta tester feedback)

- **Permission auto-approval**: Added `.claude/settings.json` with scoped allow rules for health data files and git operations. Eliminates permission prompts during normal use.
- **File renames**: All data files renamed to human-readable names visible in Claude Code UI:
  - profile.json → health-profile.json
  - session-status.json → status.json
  - lab-history.json → lab-results.json
  - treatment-stack.json → treatments.json
  - environment.json → lifestyle.json
  - provider-log.json → visit-notes.json
  - my-config.json → preferences.json
  - Schema files renamed to match.
- **Onboarding intake pacing**: Expanded structured intake with depth-aware categories. Family history, medical history, lifestyle, and medications now have multi-exchange guidance. Added "anything else?" completeness checks before transitions. Added revisit-welcome instruction (human memory is associative, not sequential).
- **Session Integrity**: New section in CLAUDE.md with four forcing functions: announce before executing, session start health check (specific data required), write verification (doctor-reads-back-chart pattern), guide application markers (evidence tiers, mechanisms).
- **Presentation rules**: Translation table for all file paths to natural language equivalents. No technical artifacts in conversation unless user asks.

## 1.0.0 — 2026-04-02

Initial release.
