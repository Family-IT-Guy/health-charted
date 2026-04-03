# Research Engine: Setup and External Dependencies

This plugin depends on files outside its own directory. This document lists
all external dependencies, where they live, and what they do.

## External Dependencies

| Dependency | Path | Purpose |
|------------|------|---------|
| Agent definition | `integrations/research-engine/agents/research-executor.md` | Custom agent for background sub-agents. Declares tools (Read, Write, Bash, Glob, Grep, WebFetch, WebSearch) and identity. |
| Settings allow rules | `~/.claude/settings.json` | `Edit(/research/**)` auto-approves Write to research/. `Bash(*pplx-curl.sh*)` auto-approves API calls. |
| API wrapper | `integrations/research-engine/scripts/pplx-curl.sh` | Perplexity API wrapper. Handles auth, directory creation, timestamps, file saving. Called with `--research "slug" 'JSON'`. |
| API key | `integrations/research-engine/.env` | Referenced internally by pplx-curl.sh. Never read directly by skills. |

## Permission Architecture

Background sub-agents cannot prompt for permissions during execution. All
permissions must be pre-approved via settings.json allow rules.

For the full permission model, evaluation order, how to add new rules, and
known limitations, see `permissions.md`.

## Perplexity API Documentation

**Source of truth**: https://docs.perplexity.ai/

Local reference files should stay current with the official docs. When API
behavior is uncertain, verify against the official documentation.

| Local file | Covers | Official docs section |
|------------|--------|----------------------|
| `api-reference.md` | Endpoints, parameters, response format, curl examples | API Reference |
| `models.md` | Model capabilities, pricing, selection guidance | Models |

### Key API details to verify periodically

- Available models and pricing (models change frequently)
- `search_context_size` nesting requirement (must be inside `web_search_options`)
- Async endpoint format for deep research
- Rate limits and retry behavior

## Version Compatibility

- **Tested on**: Claude Code v2.1.69 (March 2026)
- **Path-scoped permissions**: Fixed as of Dec 2025 (issues #15894, #21397, #13340)
- **permissionMode limitation**: `permissionMode: acceptEdits` in agent definitions
  does NOT propagate to background agents. Use settings.json allow rules instead.
  See `permissions.md` for details.
