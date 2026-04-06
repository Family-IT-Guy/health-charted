# Changelog

## 1.3.0 — 2026-04-06

### Automatic updates

The system now detects when a newer version is available and offers to update with a single yes/no confirmation. Updates back up your data first, replace only system files (guides, schemas, configuration), and never touch personal health data. If anything goes wrong, the system restores automatically.

- Added MANIFEST file listing all system files (the allowlist for updates)
- Added guides/update.md with the full update procedure
- Updated session initialization to offer updates instead of just informing

## 1.2.0 — 2026-04-06

### Knowledge layer: entity-aware compounding

The knowledge layer (reference/ and research/) now supports entity awareness, pointer-based references, change propagation, and gap detection. These changes make the system's understanding compound over time instead of being frozen at creation.

**Entity index**
- Research and reference INDEX schemas now include an `entities` field listing canonical entity names (conditions, medications, lab markers, mechanisms, interactions, symptoms, providers) covered by each file
- Research files include entities in YAML frontmatter alongside existing date_created and topic fields
- Removed review_by from research frontmatter (staleness is event-driven via propagation, not calendar-driven)

**Pointer+interpretation references**
- Reference files store pointers to source data and evidence, plus interpretations and narrative. Never copies of raw values.
- Format documented in research-guide.md Phase 4
- Interpretations are verifiable: the LLM follows pointers to check current data against recorded interpretation

**Propagation flagging**
- When my-data/ changes, the entity index identifies affected reference files
- Affected files are flagged in status.json (pending_reference_updates) for conversational review
- Flags persist across sessions until resolved
- Research changes trigger the same flagging process

**Gap detection**
- Proactive surfacing now checks entity coverage: active conditions and medications without research are surfaced as gaps
- Pending reference updates surface at session start with reason and date

**Status schema**
- Added pending_reference_updates array to status.json for propagation flag persistence

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
