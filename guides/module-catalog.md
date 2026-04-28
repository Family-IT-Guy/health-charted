# Module Catalog

A factual list of modules known to Health Charted. Each entry includes the module's name, description, repository URL, license, and maintainer. No endorsement or recommendation is implied — these are simply modules HC knows about. Users decide what fits their situation.

## When to read this guide

Read when the user asks:

- What modules can I install?
- What's available for [domain]?
- Wants to install a named module without supplying a URL

For a module the user names, look up the entry, surface the description and repo URL, ask the user to confirm before installing.

For a domain question (e.g., "what's available for fertility?"), surface entries whose description matches the domain.

If the user asks about modules not in this catalog, they may be community-published (Tier 3). Ask the user for the repo URL.

## Tier 1: Family-IT-Guy published modules

Modules curated and maintained by Family-IT-Guy. License default: CC BY-NC-SA 4.0.

_(No modules currently published. Prenatal module expected in HC v1.9 release.)_

<!-- When modules ship, add entries in this format:

## <module-name>

<one-sentence factual description>

- Repo: github.com/Family-IT-Guy/health-charted-module-<name>
- License: <SPDX or short name>
- Maintainer: Family-IT-Guy
- Current version: <semver>
- Status: <stable | beta | deprecated>

-->

## Tier 2: Registry-curated community modules

_(Registry not yet available. Roadmap item for v2 — healthcharted.com module gallery.)_

## Tier 3: Other community modules

Modules published by community authors not listed in Tier 1 or Tier 2. Discovery is via direct URL share or community channels. To install a Tier 3 module, the user supplies the repo URL.

When installing a Tier 3 module, surface to the user: "This is from a source not curated by Health Charted. I'll fetch and validate it, then show you what's inside before completing install. Proceed?"

## Adding a module to this catalog

If you've published a Tier 1 module under `Family-IT-Guy/health-charted-module-<name>`, open a PR against `Family-IT-Guy/health-charted` adding an entry above. See `CONTRIBUTING.md` for module authoring spec.

## v2 evolution

This guide is the v1 substitute for a proper module registry. v2 ships a healthcharted.com module registry with Tier 2 community curation, programmatic lookup, and a web UI. This file may be deprecated or repurposed when the registry lands.
