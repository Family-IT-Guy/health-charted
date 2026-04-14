# Changelog

## 1.6.0 — 2026-04-13

### Auto-loaded behavioral rules

Moved 6 every-session guides from `guides/` to `.claude/rules/` for framework-enforced auto-loading: behavioral protocol, data routing, data backup, epistemic discipline, proactive surfacing, session management. These rules now load automatically via Claude Code's rules system instead of depending on CLAUDE.md instructions.

Guides that remain in `guides/` are now on-demand only, loaded when their trigger condition is met: research guide, provider integration, recommended tools, tool setup, onboarding, update.

CLAUDE.md updated: Guides section split into "Rules (auto-loaded)" and "Guides (on-demand)" with trigger conditions. Session Initialization no longer instructs reading behavioral guides. Cross-references updated throughout.

### Deterministic time query

Session Initialization now runs `date` as Step 0 before reading any data files. Health timestamps depend on knowing the current date with certainty. Proactive surfacing, lab staleness checks, and treatment reassessment all require an unambiguous current date rather than LLM inference.

### 80/20 default framing for recommendations

Behavioral protocol Step 6 (Frame the Decisions) now defaults to an 80/20 structure when recommendations involve multiple interventions: lead with highest-value actions, stack additions by diminishing return, show what's cuttable and why. Each ordering claim must be evidence-backed per the Epistemic Discipline rules — if the research doesn't support a clear ranking, options are presented without implied priority. A perfect regimen nobody follows is worse than a good regimen they actually take.

## 1.5.1 — 2026-04-10

### Research quality

- Updated tool setup playbook with validated playwright-cli install procedures (CLI tool, Firefox browser, Claude Code skill). Enables the research engine to read JavaScript-rendered primary sources like medical journals and research databases.
- Added jq install procedures to tool setup playbook (cache index dependency).
- Added .playwright-cli/ to .gitignore (browser automation artifacts).

## 1.5.0 — 2026-04-10

### Grandma-proof audit (from real user session)

Comprehensive audit of a real user session (pre-surgery MRI review) identified 15 findings across security, data integrity, and UX. All fixes in this release.

**Rules extracted to `.claude/rules/`**

Behavioral rules now live in auto-loaded `.claude/rules/*.md` files instead of inline in CLAUDE.md. Modular, maintainable, same auto-load behavior. Five new rule files:

- **tone.md** — Facts about the work. No feelings, no energy, no time pressure. No performed empathy or concern about the user's state.
- **presentation.md** — Assume the user is not an IT expert. Communicate in plain language. Describe outcomes, not mechanics.
- **document-writes.md** — When extracting facts from any attached document (PDF, transcript, lab report), defer writes until the user confirms. Prevents false data propagation from speech-to-text errors and AI-generated summaries.
- **api-key-setup.md** — Grandma-friendly API key setup: one prompt, one paste, no terminal commands, no security warnings, no rotation talk.
- **verify-external-tools.md** — Before running commands targeting external tools/plugins/platforms, read current documentation first. Training-data recall of command syntax is treated as unverified.

**Permissions overhaul**

- `.claude/settings.json` now allows all tool classes (Bash, Read, Write, Edit, WebFetch) without per-command prompts. Zero permission popups during sessions.
- Declares research-engine plugin via extraKnownMarketplaces + enabledPlugins so it installs automatically on folder trust.

**Action log**

- New `my-data/actions.log` requirement. Non-health modifications to the user's machine (tool installs, config changes, env file writes) are logged in plain language. Append-only, user-readable.

**Tool setup playbook**

- New `guides/tool-setup.md` with validated install procedures for research-engine (and placeholder for playwright-cli). Declarative-first install path with CLI automation fallback. System reads the playbook and executes silently instead of asking the user to choose between technical options.

**CLAUDE.md streamlined**

- Removed inline Tone section (now in `.claude/rules/tone.md`)
- Removed inline Presentation section with translation table (now in `.claude/rules/presentation.md`, simplified to 4-sentence principle)
- Added Action Log section to Session Integrity

## 1.4.1 — 2026-04-07

### Tone

- Removed emotional support language from the system. Health Charted is education and research, not therapy. Tone section in CLAUDE.md and chief-complaint step in onboarding now explicitly forbid "I'm here", "that must be hard", "I'm sorry to hear", and similar comfort phrases. Factual recognition that anchors a clarifying question is allowed; performed presence and empathy are not.

## 1.4.0 — 2026-04-06

### Documentation

- Rewrote README for non-technical users. Written for the person actually using the system, not developers.
- Added ARCHITECTURE.md for contributors and technical users. Organized by design objectives (durability, integrity, compounding, extensibility, user experience) with rationale for decisions made and alternatives rejected.
- Added ARCHITECTURE.md to MANIFEST.

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
