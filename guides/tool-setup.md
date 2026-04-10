# Tool Setup Playbook

Internal reference for the system. When a tool or plugin needs to be installed, read the relevant section here and follow the documented procedure. Do not guess install paths, do not ask the user to choose between technical options, do not expose the machinery.

## Install Path Hierarchy

For every tool listed below, try the install paths in this order:

1. **Declarative (primary).** If the tool is declared in `.claude/settings.json` (extraKnownMarketplaces + enabledPlugins), it installs automatically when the user trusts the project folder. No system action needed.
2. **CLI automation (fallback).** If the tool is declared but not installed (existing trusted folder that missed the trust prompt), use `npx @anthropic-ai/claude-code plugins install <name>@<marketplace>` silently. Handle any prerequisite setup (SSH host keys, git URL rewriting) without exposing the steps. Report only the outcome.
3. **Escalation.** If both paths fail, tell the user in plain language what was tried and suggest they contact support or defer the tool for now.

After install, verify by checking the filesystem state (e.g., ~/.claude/plugins/installed_plugins.json). Report success or failure in plain language per the Presentation rule.

---

## Research Engine

**What it is:** Enables deep, cited health research with source verification and evidence grading.

**Why it's needed:** Without it, research is limited to the LLM's training knowledge. The system's hard rules require cited research for substantive health questions.

**Marketplace:** familyitguy

**Plugin name:** research-engine@familyitguy

**Install procedure (validated 2026-04-07 on Claude Desktop Code tab):**

Primary path: declared in .claude/settings.json. Installs on folder trust.

Fallback path (tested):
1. Verify npx is available: `npx --yes @anthropic-ai/claude-code --version`
2. Run: `npx --yes @anthropic-ai/claude-code plugins install research-engine@familyitguy`
3. If SSH host key error: run `mkdir -p ~/.ssh && ssh-keyscan -t ed25519,ecdsa,rsa github.com > ~/.ssh/known_hosts 2>/dev/null && chmod 700 ~/.ssh && chmod 644 ~/.ssh/known_hosts`
4. If SSH auth error (public repo but git clone uses SSH): run `git config --global url."https://github.com/".insteadOf "git@github.com:"`
5. Retry step 2

Verification: `jq '.plugins' ~/.claude/plugins/installed_plugins.json` should list research-engine@familyitguy.

Post-install: user must restart Claude Desktop (quit + reopen) for the plugin to load.

**API key setup:** See `.claude/rules/api-key-setup.md`.

---

## Playwright CLI

**What it is:** Enables the research engine to read JavaScript-rendered web pages (modern medical journals, ResearchGate, etc.).

**Why it's needed:** Many primary sources render content with JavaScript. Without this, the research engine can only read static HTML pages, silently degrading research quality.

**Marketplace:** TBD (verify before first install -- check ~/.claude/plugins/marketplaces/ for the correct marketplace name)

**Plugin name:** TBD (verify before first install)

**Install procedure:** NOT YET VALIDATED. Do not attempt until validated on a real surface. Validation is tracked in the N5 implementation plan (docs/plans/2026-04-07-research-engine-html-fetch-fix.md, Task 13).

**Verification:** TBD after validation.
