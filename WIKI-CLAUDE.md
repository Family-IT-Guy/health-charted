# Wiki Schema — Health Charted

The wiki is a structured catalog over the substance layer (`research/raw/`). It is pure librarian: thin catalog entries that point to sources and cross-reference each other. No substance lives in wiki pages.

The wiki layer is shipped empty in HC core. Modules populate their own wiki content (entities, concepts, sources for their domain). HC core may seed general-pharmacology and general-physiology entities over time.

## Three artifact types

All three are thin. The only original content is the one-sentence identification line.

### Entity pages (`wiki/entities/<kind>/<canonical-name>.md`)

One per canonical entity. Entity kinds live in subdirectories by type. Common kinds include: medication, supplement, hormone, lab_marker, condition, mechanism, receptor, enzyme, regulatory_entity, regulatory_pathway, clinician, lab_service, trial, side_effect, outcome. The taxonomy is open — modules introduce new kinds as their domains require.

```markdown
---
type: entity
entity_kind: <kind>
canonical_name: entity-name
submodule_path: <module:submodule:...>  # for module-shipped entities
aliases: [alias1, alias2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Entity Name

[Entity kind]. One-sentence identification.

## Where to start

[Optional: pointer to the single best source for orientation]

## Sources

- `research/raw/<filename>.md` — deep research
- `research/raw/<other-file>.md` — additional coverage

## Cross-references

- [[related-entity-1]]
- [[related-entity-2]]
- [[related-concept]]
```

### Concept pages (`wiki/concepts/<canonical-name>.md`)

One per canonical concept or mechanistic framework.

```markdown
---
type: concept
canonical_name: concept-name
submodule_path: <module:submodule:...>  # for module-shipped concepts
aliases: [alias1]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Concept Name

Concept. One-sentence definition.

## Where to start

[Optional pointer to best source]

## Sources discussing this concept

- `research/raw/<filename>.md` — [brief note on what coverage exists]

## Cross-references

- [[related-concept]]
- [[related-entity]]
```

### Source pages (`wiki/sources/<canonical-slug>.md`)

One per substance file. Describes what the source IS, not what it says.

```markdown
---
type: source
source_date: YYYY-MM-DD
ingested: YYYY-MM-DD
source_kind: deep-research|trial-publication|regulatory-document|clinician-content|...
content_type: <domain-specific-content-type>
source_path: research/raw/<filename>.md
---

# YYYY-MM-DD — <description>

One sentence describing what this source IS.

## Entities

- [[entity-1]], [[entity-2]]

## Concepts

- [[concept-1]], [[concept-2]]

## Source path

`research/raw/<filename>.md`
```

## INDEX.json

Master catalog at `wiki/INDEX.json`. Machine-parseable. Supports cross-referencing, gap detection, propagation flagging.

```json
{
  "_type": "health-charted/wiki-index",
  "entries": [
    {
      "file": "entities/<kind>/<canonical-name>.md",
      "canonical_name": "<canonical-name>",
      "entity_kind": "<kind>",
      "aliases": ["<alias>"],
      "summary": "<one-sentence summary>",
      "cross_refs": ["<related-entity>", "<related-concept>"]
    }
  ]
}
```

Modules ship their own `wiki/INDEX.json` covering their entries; HC core ships a base `wiki/INDEX.json` that's empty initially. The LLM reads the union at session start (per Session Initialization) when needed.

## Naming conventions

- Lowercase, hyphens: `metformin.md`, `hba1c.md`, `type-2-diabetes.md`
- Aliases in frontmatter handle variants
- Disambiguation when names collide: `testosterone.md` (hormone) vs `testosterone-cypionate.md` (medication delivery form, pointers back to testosterone hormone)

## Cross-reference discipline

- Use Obsidian `[[wikilink]]` syntax — human-readable, clickable in Obsidian, greppable in CLI
- Bidirectional by convention: if entity A lists entity B in cross-references, entity B should list entity A
- Reserve cross-references for meaningful relationships: mechanism-sharing, class-membership, contraindication, regulatory-precedent, monitoring-overlap, related-condition
- Within a module: unscoped wikilinks (`[[entity]]`) resolve within the module's submodule path
- Within same module, different submodule: submodule-scoped (`[[<sibling-submodule>:<entity>]]`)
- To core wiki entities: unscoped if the entity is in core
- Cross-module references: forbidden (modules are self-contained per architecture)

## Flagging (flags.json)

Event-driven attention management. A `flags.json` may live alongside `INDEX.json` to track items needing review. Created when:

- New source arrives that mentions an existing entity
- User data changes in a way that affects interpretation of a cataloged entity
- Regulatory or evidence-base shifts affect entries in the catalog

See `.claude/rules/data-routing.md` Propagation Flagging for format.

## Additive-only schema

Fields can be added to frontmatter, never renamed or removed. New entity kinds added as needed (open-ended taxonomy). A page created today must still be valid years from now.

## Ingest

When a new entity is mentioned that has no wiki entry, follow the relevant ingest guide (e.g., a module's `guides/wiki-ingest.md` if one exists). Summary: run deep research to create the substance file, then create the source page, then create or update the entity page, then wire cross-references, then update `INDEX.json`.

## What lives where

| Layer | Where | What |
|-------|-------|------|
| Substance | `research/raw/` | Full deep research files, cited evidence, the actual content |
| Catalog | `wiki/` | Thin catalog pages, pointers, cross-references, INDEX |
| Personal data | `my-data/` | User's labs, treatments, symptoms, decisions |
| Personal interpretation | `reference/` | User's pointer+interpretation files tying their data to catalog |

The wiki never contains substance. The substance layer never contains personal data. The personal layer never contains shared catalog content.
