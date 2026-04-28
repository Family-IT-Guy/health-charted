# Contributing to Health Charted Modules

This guide is for authors building modules that extend Health Charted. If you're a Health Charted user looking to install or use modules, see `guides/module-catalog.md`.

## What modules are

A module is a self-contained knowledge bundle that adds domain-specific capability to a Health Charted instance. Modules contribute:

- Wiki content (entities, concepts, source pages — domain knowledge)
- On-demand guides (reasoning patterns triggered by domain conditions)
- Schemas (for new data types the module introduces)
- my-data templates (empty starter files for new data types)
- Reference and research files (pre-loaded interpretations and evidence)
- Preference defaults (parameterize core behavior for the module's domain)

Modules cannot ship:

- Auto-loading behavioral rules (those live in HC core; if your module needs a rule, propose promotion to core)
- Overrides of core rules or other modules' content
- Executable code (no install scripts, no post-install hooks, no runtime code)

The boundary is non-negotiable. Modules are knowledge artifacts, not executable extensions.

## Quick start

1. Fork the module template repo: `github.com/Family-IT-Guy/health-charted-module-template`
2. Rename your fork: `health-charted-module-<your-domain>`
3. Edit `MODULE.json` to declare your module's metadata
4. Replace placeholder content with your domain knowledge in `wiki/`, `guides/`, `schemas/`, `research/`, `reference/`
5. Update `CHANGELOG.md` with your initial release
6. Publish to your github repo
7. (Optional) Submit to the Tier 1 catalog by opening a PR against `Family-IT-Guy/health-charted` with an entry in `guides/module-catalog.md`

For local testing during development: place your module folder directly in a Health Charted instance's `modules/<name>/` directory and start a session. The module engages on domain triggers without needing a published repo.

## File structure

```
modules/<name>/
├── MODULE.json              (required)
├── README.md                (required — plain-language description for users)
├── CHANGELOG.md             (required — dated entries per release)
├── LICENSE                  (required — your choice, no default)
├── guides/                  (optional)
│   └── <guide-name>.md
├── schemas/                 (optional)
│   └── <type-name>.schema.json
├── my-data-templates/       (optional)
│   └── <type-name>.json
├── wiki/
│   ├── INDEX.json
│   ├── entities/
│   ├── concepts/
│   └── sources/
├── reference/
│   ├── INDEX.json
│   └── *.md
└── research/
    ├── INDEX.json
    └── raw/
        └── <YYYY-MM-DD>-<topic>.md
```

## MODULE.json

The canonical declaration of your module. Required fields:

- `name`: your module's short identifier (lowercase, hyphenated)
- `version`: semver (e.g., `"1.0.0"`)
- `namespace`: how your module's wiki entities are scoped (typically same as `name`)
- `description`: one-sentence factual description
- `license`: SPDX identifier or short name
- `requires_hc_min_version`: minimum HC core version your module needs

Optional fields:

- `author`: object with `name` and `url`
- `repository`: object with `type` and `url`
- `homepage`: URL
- `domain_triggers`: text describing when this module is generally relevant (advisory hints to the LLM, not regex)
- `salience_hints`: text describing state-based observations that may warrant module reasoning
- `guides`: array of `{file, trigger}` objects declaring on-demand guides
- `submodules`: array of submodule declarations (recursive, see below)

Example minimal MODULE.json:

```json
{
  "name": "prenatal",
  "version": "1.0.0",
  "namespace": "prenatal",
  "description": "Prenatal nutrition and pregnancy-stage physiology, lab markers, common conditions.",
  "license": "CC-BY-NC-SA-4.0",
  "requires_hc_min_version": "1.8.0"
}
```

Example full MODULE.json with submodules and triggers:

```json
{
  "name": "hormones",
  "version": "2.0.0",
  "namespace": "hormones",
  "description": "Hormone pharmacology, mechanism walks, evidence discipline, optimization patterns.",
  "license": "CC-BY-NC-SA-4.0",
  "requires_hc_min_version": "1.8.0",
  "author": { "name": "Author Name", "url": "https://..." },
  "repository": { "type": "git", "url": "https://github.com/Family-IT-Guy/health-charted-module-hormones.git" },
  "domain_triggers": "Hormone-related questions: testosterone, estrogen, progesterone, thyroid, cortisol, etc.",
  "salience_hints": "User data referencing hormone-related entities or labs may warrant module reasoning.",
  "guides": [
    { "file": "guides/hormone-evidence-discipline.md", "trigger": "When the user asks about a specific hormone or its evidence base" }
  ],
  "submodules": [
    {
      "namespace": "testosterone",
      "version": "1.5.0",
      "description": "Testosterone-specific entities, mechanisms, and protocols.",
      "domain_triggers": "Mentions of testosterone, T levels, free-T, total-T, SHBG, TRT.",
      "salience_hints": "User data referencing testosterone-related entities or labs.",
      "intake_questions": [
        "Are you currently on or considering TRT?",
        "Have you had recent total-T, free-T, or SHBG labs?"
      ],
      "covers_entities": ["testosterone", "free-testosterone", "shbg", "androgen-receptor"],
      "submodules": []
    }
  ]
}
```

## Wiki entries

Each wiki entity, concept, and source file uses YAML frontmatter:

```yaml
---
type: entity
canonical_name: testosterone
submodule_path: hormones:testosterone
created: 2026-XX-XX
updated: 2026-XX-XX
aliases: [test, T]
---

# Entity content...
```

Required:

- `type`: one of `entity`, `concept`, `source`
- `canonical_name`: stable identifier, hyphenated lowercase
- `submodule_path`: full namespace path (`module:submodule:subsubmodule`)

Optional:

- `aliases`: alternate names
- `created`, `updated`: ISO dates
- `entity_kind`: subtype categorization (e.g., `medication`, `condition`, `lab-marker`)

## Cross-references

Use wikilinks (`[[entity-slug]]`) for references between entities.

- Same submodule: unscoped (`[[testosterone]]` resolves within the current submodule)
- Same module, different submodule: submodule-scoped (`[[hcg:beta-hcg]]`)
- Across modules: forbidden. Modules are self-contained. If shared content is genuinely needed, propose promoting it to HC core (which all modules can reference unscoped).
- To HC core wiki: unscoped if the entity is in core (`[[receptor-dynamics]]`)

The LLM resolves wikilinks by searching scopes in order: same submodule, ancestor submodules, core wiki. Unresolved wikilinks fail validation at install.

## The Module Promotion Filter

Anything you state as "domain X has property Y" is empirical, not principled. It's time-bound and will become stale. A principle has the form: **[universal mechanism or epistemic constraint] → [behavior]**.

When deciding what content goes in your module vs. what should be promoted to HC core, run this filter:

1. Strip the domain-specific claim
2. State the reasoning the claim was supporting
3. Express the reasoning as a domain-independent principle
4. The principle promotes to HC core (propose via PR); the domain claim becomes an *example* illustrating the principle in your module's documentation

The filter catches: empirical-claim disguised as principle, appeals to authority dressed as evidence ranking, industry/community heuristics with implicit assumptions, time-bound state-of-the-research statements.

If your module wants behavior that needs to be globally active across all HC sessions (not just yours), the path is to propose the principle to HC core, not to embed it as a module rule. Modules cannot ship rules.

## Versioning

Modules use semantic versioning (semver). Each release bumps the version per these rules:

- **PATCH** (1.0.X): typo fixes, content corrections, citation updates
- **MINOR** (1.X.0): additive content (new entities, new guides, new submodules)
- **MAJOR** (X.0.0): breaking changes (renamed entities user data references, removed content other content depended on, schema changes that break existing user data)

Follow the additive-only commitment for minor releases: don't rename entities, don't remove existing content, don't break existing wikilinks. Breaking changes always bump major.

Document every release in `CHANGELOG.md` with a dated entry, grouped by submodule when relevant.

Tag releases in your git repo (`v1.0.0`, `v1.1.0`) for deterministic install.

### Submodule versioning

Each submodule has its own version, independent of the parent module's version. A submodule's PATCH/MINOR/MAJOR semantics apply within that submodule's content scope. Parent module version doesn't auto-cascade from submodule changes — bump it separately when module-wide structure changes (taxonomy, top-level guides, top-level entities) or when you want a coordinated release marker.

## Submodules

Submodules are recursive logical groupings within a module. Use them when:

- A subset of your module has its own evolution cadence (e.g., testosterone updates frequently while hcg updates rarely)
- A subset has its own engagement granularity (LLM should engage just `hormones:testosterone` rather than all of `hormones`)
- You want to declare submodule-specific intake questions or salience hints

Don't use submodules when:

- A submodule has only one entity (flatten it into the parent)
- A single-leaf chain (a:b:c:single-entity) emerges (signals over-decomposition)

There is no architectural cap on submodule depth. Author judgment applies.

## License

Declare your license explicitly in `LICENSE` and in `MODULE.json` `license` field. Health Charted core uses CC BY-NC-SA 4.0; modules may use any compatible license including more permissive (MIT, Apache 2.0) or more restrictive (proprietary, custom).

There is no default. Modules without a declared license fail validation.

## Local development

To test a module locally without publishing:

1. Place your module folder directly in a Health Charted instance: `<HC-instance>/modules/<your-module>/`
2. Start a session in that instance
3. The session-init module scan picks up your module
4. Engage it via domain triggers in conversation

Iterate: edit module files, start a fresh session to pick up changes (since modules are scanned at session start, not hot-reloaded mid-session).

## Publishing

For v1, modules are published as standalone github repos. Users install by URL.

To list your module in the Tier 1 catalog (for FITG-published modules), open a PR against `Family-IT-Guy/health-charted` adding an entry to `guides/module-catalog.md`.

For community modules: publish to your own github repo. Share the URL with users who want to install. Future versions of Health Charted will support a registry on healthcharted.com for broader discovery.

## Validation at install

The Health Charted system validates these properties when a module is installed or loaded:

- MODULE.json parses
- Required fields present (`name`, `version`, `namespace`, `description`, `license`, `requires_hc_min_version`)
- `version` is valid semver
- `requires_hc_min_version` ≤ current HC core VERSION
- All declared `guides[].file` paths exist
- All declared schemas in `schemas/` exist
- `covers_entities` slugs exist in `wiki/INDEX.json`
- Wiki entities' `submodule_path` values match a declared submodule namespace path
- Submodule versions are valid semver
- Internal wikilinks resolve

Validation failures surface to the user in plain language and block install or load. Modules with validation issues should be fixed before publishing.

## Questions or proposals

If you've got content you think should promote to HC core (per the Module Promotion Filter), open an issue or PR against `Family-IT-Guy/health-charted` with the proposed principle and reasoning. The filter is the bar; promotions land in HC core's `.claude/rules/` and benefit all modules.

For module-specific questions, use your module repo's issues. For HC core questions, use `Family-IT-Guy/health-charted` issues.
