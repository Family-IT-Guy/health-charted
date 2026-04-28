# Changelog

## 1.8.0 — 2026-04-28

### Modular architecture

Health Charted gains a module system. Modules are self-contained knowledge bundles (wiki entries, on-demand guides, schemas, my-data templates, reference and research files) that plug into a Health Charted instance and engage dynamically when the user's domain triggers them. Modules cannot ship behavioral rules — those live only in HC core. See `CONTRIBUTING.md` for the full module specification.

- `session-initialization.md` gains step 6: scan installed modules in `modules/<name>/MODULE.json`, validate `requires_hc_min_version`, build the discovery layer (namespace, triggers, hints, declared guides, recursive submodule tree). Module content (wiki, research, reference, guides) loads only on domain trigger.
- `CLAUDE.md` adds a Modules section explaining module discovery and dynamic loading, plus a `guides/module-catalog.md` row in the on-demand guides table.
- New `CONTRIBUTING.md` documents the canonical module spec: file layout, `MODULE.json` schema (with recursive submodules and per-level versioning), wiki frontmatter conventions, cross-reference rules, the Module Promotion Filter, semver discipline, license requirements, local development workflow, publishing flow.
- New `guides/module-catalog.md` lists known Tier 1 (FITG-published) modules. v1 substitute for a registry; healthcharted.com module gallery is v2 roadmap.
- Companion module template scaffold lives outside this repo at `health-charted-module-template`.

### Rule promotions and sharpenings

Eight new universal principles promoted from the peptide prototype's discipline analysis. Each was an empirical or domain-specific framing in the source rule; sharpened into a domain-independent principle here.

- **Domain Projection Discipline** (`epistemic-discipline.md`) — evidence collected in one domain (species, population, condition, dose range, timeframe) cannot be projected to another without explicit mechanism-traceable reasoning. Each projection step loses confidence.
- **Combination Evidence Discipline** (`epistemic-discipline.md`) — combinations have their own evidence requirements separate from their components. Component evidence does not transfer to the combination.
- **Confidence-Authority Decoupling** (`epistemic-discipline.md`, replaces "Handling Institutional Positions") — confidence in a claim tracks underlying evidence quality on multiple axes; institutional and regulatory status is a distinct axis. Approval is regulatory fact, not scientific fact.
- **Threshold-as-Claim Discipline** (`hard-rules.md` and `epistemic-discipline.md`, replaces the standalone functional-range rule) — all reference ranges (standard, functional, context-specific) are interpretation thresholds with their own evidence bases. Surface what supports a threshold rather than smuggling in "this is the right number."
- **Severity-Triggered Surfacing** (`hard-rules.md` new bullet, `proactive-surfacing.md` new top trigger) — severity classes that require immediate user action trigger explicit, prominent surfacing — never burial in routine output.
- **Loaded-Language Translation** (`communication.md`) — user-supplied vocabulary often encodes assumptions about mechanism, evidence, or value that become visible only when restated mechanistically. Acknowledge what the user means and translate to mechanism-level terms before engaging substantive reasoning.
- **Schedule-as-Hypothesis** + **Phase-Distinct Calibration** (new `treatment-dynamics.md` rule file) — published intervention schedules describe what was tested, not what is universally optimal; treat them as starting hypotheses calibrated against this user's response. Different phases (induction, maintenance, discontinuation, cycling) require their own mechanism analysis. Adaptation, plateau, and failure all look like "no change" but have different mechanism causes.

Two rules promoted from the peptide prototype to HC core, sharpened along the way:

- **Consultative Posture** (`.claude/rules/consultative-posture.md`) — how to interact when the user is exploring outcomes, goals, or trade-offs. Foundation interventions reframed as baseline modulators (not "largest effect sizes" empirical claim). Domain-specific examples stripped. Cross-references the new core principles.
- **Trajectory Analysis** (`.claude/rules/trajectory-analysis.md`) — slope > point, minimum data for a trend claim, rate of change, subjective-objective integration, variability as signal. Functional-range framing aligned with Threshold-as-Claim.

### Evidence Tier table replaced with axis-based framework

The old Quality Tiers table (Tier 1 RCT through Tier 5 anecdotal) is removed from `epistemic-discipline.md`. Replaced with a multi-axis evaluation framework: methodology rigor, replication, mechanism traceability, conflict-of-interest exposure, generalizability. Typical patterns by evidence type (RCT, observational, case report, mechanism, anecdote) are documented but not as ranked tiers — no category gets default confidence based on category alone.

This catches a real failure mode where tier shorthand ("Tier 1 evidence") was being treated as a confidence default that bypassed actual evaluation. Aligns with the Confidence-Authority Decoupling principle.

### Step 6 of behavioral protocol: Synthesize and Propose

Step 6 renamed from "Frame the Decisions" (option-menu philosophy) to "Synthesize and Propose." Where evidence points to a specific path given the user's context, the framework synthesizes a concrete proposal with reasoning shown — mechanism, evidence quality, user-context factors, tradeoffs — rather than presenting an option menu. Users can override via `preferences.json` `decision_framing: "menu"` field. New "User modifies, framework tracks" subsection routes accepted plans to `my-data/strategic-plans.json` and `my-data/treatments.json`.

### Tool-specific content moved to guides

Insulates rules from tool churn so principles stay stable across decades.

- `hard-rules.md` research-engine references parameterized to "configured deep-research tool" (referring users to `guides/recommended-tools.md` for current selection).
- `api-key-setup.md` Perplexity-specific script moved to `guides/tool-setup.md` Research Engine section. Rule now states the conversational pattern (request key plainly, save without exposing technical detail) with no tool-specific names.
- `data-backup.md` tool-specific content (Mac Time Machine, Windows File History, Backblaze, git commands) moved to new `guides/backup-setup.md`. Rule now states only the principles (auto-versioning, external backup with version history required, sync-only insufficient).

### presentation.md persona refactor

Removed hardcoded "user is not an IT expert" assumption. New version adapts to demonstrated user fluency: communicate technical detail in plain language unless the user has demonstrated technical fluency on the topic.

### Anecdote handling

Closing rationale paragraph removed from `epistemic-discipline.md` "Handling Anecdotes" section. The rule is purely epistemic; values-adjacent framing about "respecting experience" was orthogonal and is gone. Procedural numbered list (don't dismiss, ask mechanism, trace pathway, document hypothesis or alternative explanation) preserved unchanged.

### Data layer additions

Promoted from peptide prototype to core for general use across modules:

- New schemas: `photos.schema.json`, `training.schema.json`, `strategic-plans.schema.json` (renamed from singular for filename convention; namespace cleaned to `health-charted/`).
- New my-data templates: `photos.json`, `training.json`, `strategic-plans.json` (empty starters with `_type: health-charted/<name>`).
- `data-routing.md` gains routing table rows for Photos, Training, and Strategic plans.
- `session-initialization.md` step 5 my-data file list extended to include the three new files.

### Wiki layer added to core

Empty wiki structure: `wiki/INDEX.json`, `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`. New `WIKI-CLAUDE.md` schema doc at root documents the librarian-model wiki: entity/concept/source page formats, INDEX format, naming conventions, cross-reference discipline, additive-only schema, what-lives-where layer separation.

The wiki ships empty in v1.8. Modules populate their own wiki content. HC core may seed general-pharmacology and general-physiology entities over time.

### Update flow parameterization

Remote VERSION fetch in `session-initialization.md` parameterized to point at `guides/update.md` for the configured source URL. Insulates the rule from any future repo move.

### Guides updated

- `research-guide.md` Phase 2 epistemic discipline reference expanded to name the new principles (Confidence-Authority Decoupling, Domain Projection, Combination Evidence, Threshold-as-Claim) and the multi-axis evaluation framework.
- `onboarding.md` adds a "Module engagement intake" section. Core onboarding stays general; module-specific intake fires on first engagement after core onboarding completes. Per-module engagement state tracked in `my-data/status.json` `engaged_modules`.

### MANIFEST

Updated to include all new system files: CONTRIBUTING.md, WIKI-CLAUDE.md, guides/module-catalog.md, guides/backup-setup.md, .claude/rules/consultative-posture.md, .claude/rules/trajectory-analysis.md, .claude/rules/treatment-dynamics.md, schemas/photos.schema.json, schemas/training.schema.json, schemas/strategic-plans.schema.json.

### Backward compatibility

Existing v1.7.0 installs upgrade additively. No file types removed, no schema changes break existing user data, no rules removed (only sharpened or refactored). Users who relied on the old "Frame the Decisions" Step 6 option-menu framing can opt back via `preferences.json` `decision_framing: "menu"`.

## 1.7.0 — 2026-04-15

### CLAUDE.md modularization

CLAUDE.md reduced from 183 lines to ~20 by extracting behavioral content into auto-loaded `.claude/rules/` files. One concept, one file.

Five new rule files:

- **communication.md** — Honesty-is-default framing, language level calibration, interaction style.
- **hard-rules.md** — Non-negotiable constraints (no advice without context, no fabrication, no appeals to authority, research-engine routing, etc.).
- **session-initialization.md** — Read-order sequence and onboarding check.
- **session-integrity.md** — Announce-before-executing, session start health check, action log, guide application markers.
- **structure.md** — Data files inventory, knowledge file layout, schemas pointer.

Two rule extensions:

- **epistemic-discipline.md** gained a "Model Limitations and Meta-Bias" section (was CLAUDE.md Transparency About Model Limitations).
- **data-routing.md** gained a "Write Confirmation" subsection capturing the "read-back IS the confirmation" framing and natural-language example (was CLAUDE.md Write verification).

CLAUDE.md now contains only the identity statement, the on-demand guides table, and a one-line pointer to auto-loaded rules. No behavioral change for users — auto-loaded rules land in context every session regardless of where they live.

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
