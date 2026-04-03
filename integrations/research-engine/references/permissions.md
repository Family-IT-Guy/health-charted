# Research Engine: Background Agent Permissions

How Write permissions work for background sub-agents dispatched by :orchestrate.

## Architecture

Background sub-agents use TWO permission mechanisms working together:

### 1. Settings.json Allow List (the actual Write fix)

In `~/.claude/settings.json`, the `permissions.allow` array includes:

```json
"Edit(/research/**)"
```

This auto-approves Write/Edit operations to `<project-root>/research/`
for ALL agents (foreground and background). The `Edit` rule covers both the
Edit and Write tools per Claude Code docs: "Edit rules apply to all built-in
tools that edit files."

**Why this works for background agents:** Permission evaluation order is
deny → ask → allow → permission mode. Allow rules are checked BEFORE the
permission mode step. Background agents auto-deny anything not pre-approved,
but a matching allow rule counts as pre-approved.

**Path pattern:** `/research/**` is project-root-relative (single
leading slash). It matches `<whatever-project-you're-in>/research/**`.
User-level settings apply globally across all projects.

### 2. Custom Agent Definition (identity + tool declarations)

The `research-executor` agent at `integrations/research-engine/agents/research-executor.md`
provides:
- Explicit `tools:` field (Read, Write, Bash, Glob, Grep, WebFetch, WebSearch)
- `permissionMode: acceptEdits` (documented but does NOT propagate to
  background agents — see Known Limitations)
- Clear identity so the orchestrator dispatches the right agent type

### 3. Bash Allow List (pre-existing)

In `~/.claude/settings.json`:

```json
"Bash(*pplx-curl.sh*)"
```

Auto-approves Bash commands containing "pplx-curl.sh" (the Perplexity API
wrapper).

```json
"Bash(*yt-dlp*)"
```

Auto-approves Bash commands containing "yt-dlp" (YouTube transcript and
metadata extraction). Used by research sub-agents to ingest YouTube video
transcripts as primary sources. Install yt-dlp: `pip install yt-dlp` or
`brew install yt-dlp`.

## Adding New Permissions

If a future skill or feature needs additional background agent permissions:

1. **Identify the tool and path** — what tool (Write, Edit, Bash) and what
   file paths does the background agent need access to?

2. **Add a scoped allow rule** to `~/.claude/settings.json` under
   `permissions.allow`. Use the narrowest scope possible:

   | Pattern | Scope |
   |---------|-------|
   | `Edit(/research/**)` | Project-root-relative directory |
   | `Edit(//absolute/path/**)` | Absolute path (double slash) |
   | `Edit(~/home/path/**)` | Home-directory-relative |
   | `Bash(*script-name*)` | Bash commands matching glob |

3. **Test empirically** — dispatch a background sub-agent and verify the
   operation succeeds. Do NOT trust documentation alone (see Known Limitations).

4. **Document here** — add the new rule to the Current Rules section below.

## Current Rules

| Rule | Location | Purpose |
|------|----------|---------|
| `Edit(/research/**)` | `~/.claude/settings.json` | Write thread files, source files, cascade YAMLs |
| `Bash(*pplx-curl.sh*)` | `~/.claude/settings.json` | Perplexity API calls and PDF downloads |

## Known Limitations

### permissionMode Does NOT Propagate to Background Agents

**Tested 2026-03-05, Claude Code v2.1.69.**

The `permissionMode: acceptEdits` field in custom agent definitions
(`~/.claude/agents/*.md`) does NOT auto-approve Write operations for
background sub-agents (`run_in_background: true`). It works only for
foreground sub-agents.

Background agents use a pre-launch approval + auto-deny mechanism that
ignores the agent definition's `permissionMode`. The settings.json allow
list bypasses this because allow rules are evaluated before the permission
mode check.

**Evidence:**
- Foreground dispatch with `permissionMode: acceptEdits`: prompted user
  (did not auto-approve)
- Background dispatch with `permissionMode: acceptEdits`: Write denied
- Background dispatch with `Edit(/research/**)` allow rule: Write
  succeeded

### Path-Scoped Patterns Had Bugs (Fixed)

GitHub issues #15894, #21397, #13340 reported that path-scoped Edit/Write
patterns failed due to relative vs absolute path mismatches. Issue #15894
was closed as COMPLETED on Dec 31, 2025. Empirically confirmed working in
v2.1.69 (March 2026).
