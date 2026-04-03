# Health Charted

Open-source medical research system that helps people understand and manage their health from first principles using LLMs.

**Website:** healthcharted.com
**License:** CC BY-NC-SA 4.0

## What It Is

A folder of files that, when pointed at by an LLM (Claude, ChatGPT, Gemini, Codex), becomes a personal health research companion. No server, no database, no account. Your data files live on your machine. The LLM is the application; the files are the state.

This system was built for Claude (the `CLAUDE.md` filename is a Claude convention). If using a different LLM, rename `CLAUDE.md` to match your platform's convention. The content is provider-agnostic and works with any capable LLM.

## The Problem

People facing health conditions are stuck between two bad options: trust institutional gatekeepers who have 15 minutes per appointment, or wade through unstructured internet noise. LLMs can reason on medical literature, but without structure, the interaction is one-shot and disposable. No accumulation, no tracking, no continuity.

Health Charted gives the LLM a structured knowledge base and behavioral protocol so it becomes an ongoing research partner that gets smarter about your situation over time.

## How It Works

### Behavioral Protocol
Every interaction follows the same sequence:
1. **Understand before advising** — validate the situation, ask clarifying questions
2. **Update data files** — capture information in real-time
3. **Interpret** — compare to condition-specific optimal ranges, trend analysis against your history
4. **Recommend** — concrete, actionable, with mechanistic reasoning
5. **Offer depth** — go deeper on any point, with references to primary sources

### Research Bootstrapping
When you start with a new health topic, the system helps you build a knowledge base:
1. **Broad landscape** — exploratory queries to map the territory
2. **Domain-specific deep dives** — mechanism, immune/inflammatory, nutritional, environmental, treatment, diagnostics, evidence quality
3. **Synthesis into reference** — deep research distilled into compact, actionable reference files
4. **Data layer** — your personal profile, history, current state, tracking

### Epistemic Discipline
- Evidence quality tiers (RCT > observational > case series > mechanistic reasoning > anecdotal)
- Hypothesis tracking with explicit validation/invalidation
- Anecdotes explored mechanistically, not dismissed
- No appeals to authority — mechanisms, not consensus
- Assumptions flagged, stale hypotheses prohibited

### Data Integrity
- All data files are JSON with formal schemas for structural consistency
- Automatic local git versioning protects against write errors (invisible to the user)
- Write verification after every data file change
- Research files include aging metadata so stale knowledge is flagged for review

## Structure

```
health-charted/
├── CLAUDE.md                # Core behavioral protocol (updatable)
├── my-config.json           # Your overrides (never touched by updates)
├── VERSION
├── CHANGELOG.md
├── LICENSE
├── my-data/                 # Your personal data (never touched by updates)
│   ├── profile.json         # Identity, conditions, providers, medications
│   ├── session-status.json  # Current priorities, action items, onboarding state
│   ├── lab-history.json     # Lab results with context and interpretation
│   ├── symptoms.json        # Symptom tracking with pattern analysis
│   ├── treatment-stack.json # Current treatments, medications, supplements
│   ├── environment.json     # Lifestyle factors, triggers, diet, sleep, stress
│   ├── provider-log.json    # Visit summaries, care team
│   └── decisions.json       # Decision log with reasoning (never pruned)
├── schemas/                 # JSON Schema definitions (updatable)
│   ├── profile.schema.json
│   ├── session-status.schema.json
│   ├── symptoms.schema.json
│   ├── lab-history.schema.json
│   ├── treatment-stack.schema.json
│   ├── environment.schema.json
│   ├── provider-log.schema.json
│   ├── decisions.schema.json
│   ├── config.schema.json
│   ├── reference-index.schema.json
│   └── research-index.schema.json
├── reference/
│   └── INDEX.json           # Populated during research bootstrapping
├── research/
│   └── INDEX.json           # Built over time
├── guides/
│   ├── behavioral-protocol.md
│   ├── data-routing.md
│   ├── data-backup.md
│   ├── epistemic-discipline.md
│   ├── proactive-surfacing.md
│   ├── provider-integration.md
│   ├── research-guide.md
│   ├── session-management.md
│   └── recommended-tools.md
└── integrations/
    └── research-engine/     # Perplexity Sonar API research plugin
```

## Design Principles

- **Data integrity and availability.** Health data accumulated over years is irreplaceable. The system uses JSON for structural consistency, git for version history, write verification for every change, and guides users through external backup setup. Data protection is a core responsibility, not an afterthought.
- **Providers are tools in your toolkit.** Not gatekeepers. You are the integrator who coordinates across specialists. The system helps you prepare for and process each provider interaction.
- **Education, not medical care.**
- **You are the decision maker.** The system presents evidence, analysis, options, and recommendations. You decide.

## Versioning and Upgrades

The framework layer (CLAUDE.md, guides/, schemas/) is updatable. Your data layer (my-data/, reference/, research/) is never touched by updates. Schema evolution is additive only: new fields are added, existing fields are never renamed or removed. When a new version adds a field, the LLM adds it with a null value the next time it writes to that file. No migration scripts, no breaking changes.

`my-config.json` holds your customizations so CLAUDE.md can be updated without losing your preferences.

## User Scenarios

- Individual managing their own health
- Parent managing a child's health
- Family health manager tracking multiple family members (one folder per person)
- Medical provider tracking patients (roadmap)

## License

Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)

You can use, adapt, and share this system freely as long as you credit the project, don't use it commercially, and share adaptations under the same license.
