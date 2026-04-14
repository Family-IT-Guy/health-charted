# Research Guide

How to build a knowledge base for a health topic. Used during onboarding and any time the user wants to research a new area.

## Before Starting

### Check existing coverage

Before researching, check what already exists in `reference/` and `research/`. If the topic is partially covered, identify gaps rather than re-researching from scratch. If coverage exists but is old, note the age and research for updates.

### Determine what to research, not what the user knows

Ask enough to know the TOPIC (condition name, symptom cluster, optimization area). Do not ask what the user already knows about the topic before researching. Their existing understanding may be incomplete or shaped by provider opinions and institutional positions. Research from first principles first.

After the research is complete, ask what they already knew, what their providers have told them, and what surprised them. Use the delta between the research and their prior understanding to guide follow-up conversation and targeted deeper research.

## Research Tools

This system uses the research-engine plugin (installed separately) for structured, cited research via the Perplexity Sonar API with academic and web search modes. If the plugin is installed and the API key is configured, use it as the primary research tool.

If the research engine is not installed, inform the user and recommend setup per `guides/recommended-tools.md`.

## Research Plan

Before starting research, present a plan organized by domain. The user approves, modifies, or reorders priorities before any research is executed.

### Domain structure

Research systematically by biological or clinical domain. Each domain becomes its own file in `research/`. Common domains (adapt to the topic):

- **Mechanism and pathophysiology** — what's happening at the molecular, cellular, or systems level
- **Immune and inflammatory factors** — how the immune system is involved
- **Hormonal and endocrine factors** — hormonal influences and interactions
- **Nutritional and metabolic factors** — nutrients, deficiencies, metabolic pathways
- **Environmental factors** — toxins, lifestyle, exposures, triggers
- **Treatment mechanisms** — how each intervention works (mechanism of action, not just efficacy claims)
- **Measurement and diagnostics** — what to test, how to interpret, optimal vs standard ranges
- **Evidence landscape** — what's well-supported vs preliminary vs contested for this topic

Not every topic needs every domain. Let the topic guide the structure. A general optimization project might need 3-4 domains. A complex condition might need all of them.

### Depth tiers

For each domain, recommend a depth:

- **Quick scan** — landscape overview, key findings, major sources. Enough to know what exists and what matters.
- **Deep dive** — full mechanistic tracing, primary source verification, pathway analysis, evidence quality assessment. Used for domains central to the user's situation.

Present recommendations but let the user choose. They know what matters most to them.

## Research Execution

### Phase 1: Broad landscape

Start with a broad exploratory pass across the topic. The goal is to map the territory before going deep:

- What is this condition or health area? What are the fundamental mechanisms?
- What body systems are involved?
- What are the primary controversies or areas of disagreement?
- What are the main approaches and their mechanistic basis?

Summarize findings and present to the user before proceeding to deep dives.

### Phase 2: Domain-specific deep dives

For each approved domain:

1. Research the domain using available research tools (Perplexity integration preferred, web search as fallback).
2. For deep dives: verify primary sources. Read the actual studies, not just summaries. Check methodology, sample size, funding, and whether conclusions follow from data.
3. Write findings to `research/[NN-domain-name].md` with numbered file names for reading order.
4. Apply epistemic discipline from the Epistemic Discipline rules: trace mechanisms, evaluate evidence quality, note institutional positions vs data, flag gaps honestly.
5. Extract entities from the research and add to the file's YAML frontmatter:

   ```yaml
   ---
   date_created: 2026-04-05
   topic: Iron metabolism and absorption
   entities:
     conditions: [iron-deficiency, anemia]
     lab_markers: [ferritin, transferrin-sat, hemoglobin, serum-iron, TIBC]
     medications: [ferrous-bisglycinate, ferrous-sulfate]
     mechanisms: [hepcidin-regulation, calcium-iron-competition, vitamin-c-enhancement]
     interactions: [calcium-iron, vitamin-c-iron, ppi-iron]
   ---
   ```

   Use canonical names from the naming discipline (data-routing.md). Entity types: conditions, lab_markers, medications, mechanisms, interactions, symptoms, providers. Add other types as needed.

6. Update `research/INDEX.json` with the file entry including the same entities.

### Phase 3: Identify gaps and cascade

After completing the planned domains, assess what's missing:

- Did any domain raise questions that another domain should address?
- Are there sub-topics that need their own focused research?
- Did sources disagree on something that needs resolution?

Present gaps to the user. If they want to pursue them, research those as additional threads.

### Updating Existing Research

Research files are not write-once artifacts. When new evidence arrives (new studies, user brings new information, propagation flag triggers review), update the existing research file rather than creating a new one.

**When to update vs create new:**
- **Update** when the new information covers the same domain as an existing file (new study on iron absorption goes into the existing iron metabolism file).
- **Create new** when the topic is genuinely new (a condition not previously researched, a domain not covered by any existing file).

**Update procedure:**
1. Read the existing research file in full.
2. Add new findings as a new section or integrate into existing sections.
3. If new findings contradict existing content, note the contradiction explicitly with dates and evidence quality for both the old and new claims. Do not silently replace.
4. Update the YAML frontmatter: add new entities if the update covers entities not previously listed.
5. Update `research/INDEX.json` entities to reflect the new coverage.
6. Check entity index: are there reference files that cite this research file? If so, flag them for interpretation review (see data-routing.md, Propagation Flagging section).

### Phase 4: Synthesize into reference

Distill the research files into reference files in `reference/`. Reference files are the interpretation layer. They contain pointers and analysis, never copies of raw data or evidence.

#### Reference file format

Each section in a reference file follows this structure:

```markdown
## [Topic]

**Sources:**
- Data: my-data/[file].json ([which entries])
- Evidence: research/[file].md ([which sections])

**Interpretation:**
[What the current data means in context of the evidence.
This is the only original content in the section.
Do not copy values from my-data/ or findings from research/.
State what the combination means for this person.]

**Narrative:**
[Chronological story of how this has evolved. Each entry is
a dated observation referencing the source data, not a copy.
Append new entries. Never modify old entries.]
- YYYY-MM: [what happened, referencing source]
- YYYY-MM: [what happened, referencing source]

**Next action:**
[What to do and when. Recheck dates, protocol adjustments,
provider discussion points.]

**Cross-references:**
- [Related topic]: see [other-reference-file.md]
```

#### Why pointers, not copies

If a reference file says "ferritin: 18 ng/mL," that value exists in two places. When lab-results.json updates, the reference is stale until manually propagated. With pointers, the reference says "marker: ferritin, source: my-data/lab-results.json." The LLM follows the pointer to get the current value. The interpretation may be stale, but the facts the LLM reads through pointers are always current.

#### Reference INDEX entry

When creating a reference file, add its entry to `reference/INDEX.json` including:
- `entities`: canonical entity names covered

Provenance (which source files the reference draws from) lives in the **Sources** section of the reference file body, not in the INDEX. The entity index handles propagation lookup.

### Phase 5: Update indices

After creating or updating files, update:
- `research/INDEX.json` — file name, contents summary, when to reference it, entities covered
- `reference/INDEX.json` — file name, contents summary, when to reference it, entities covered

## Research Quality Standards

All research follows the epistemic discipline in the Epistemic Discipline rules. In particular:

- Trace mechanisms, not authorities
- Evaluate methodology, not just conclusions
- Distinguish between what data shows and what institutions claim
- For deep dives, read primary sources rather than relying on summaries
- Flag gaps honestly — "this hasn't been studied" is more useful than speculation

## Ongoing Research

The knowledge base is not static. As the user's situation evolves:

- New symptoms or concerns may require new research domains
- Treatment options not previously explored may become relevant
- New evidence may update previous understanding
- Open hypotheses may need targeted investigation

When the system identifies a knowledge gap during normal interaction, suggest targeted research to fill it. Use this guide for the approach.

## Research Frontmatter

Every research file must include YAML frontmatter:

```
---
date_created: 2026-04-02
topic: Dupuytren's contracture treatment mechanisms
entities:
  conditions: [dupuytren-contracture]
  mechanisms: [myofibroblast-proliferation, palmar-fibromatosis]
  medications: [collagenase-clostridium, xiaflex]
  lab_markers: []
---
```

`date_created` records when the research was written. It's a fact, not a trigger. Research staleness is detected through entity-based propagation (new data or evidence arrives for the same entities), not through calendar dates.
