# Recommended Tools

Tools that enhance the system. None are required to start using Health Charted, but each improves the quality of specific capabilities. These are surfaced to the user on first need, not as a prerequisite.

## Research Engine Plugin (strongly recommended)

Cited web research via the Perplexity Sonar API. Background sub-agents, primary source verification, cascade follow-ups.

**What it does:** Enables deep, citeable research with source verification and evidence grading. Without it, research is limited to whatever the LLM platform provides natively.

**When to surface:** On first research need. If the user asks to research a health topic and the research-engine plugin is not installed, pause and offer setup.

**Prerequisites:** jq (JSON processor). Check if installed by running `jq --version`. If not installed, walk the user through installation before proceeding:

- **macOS with Homebrew:** `brew install jq`
- **macOS without Homebrew:** Download the binary from https://github.com/jqlang/jq/releases (choose the macOS Apple Silicon or Intel version), move it to `/usr/local/bin/jq`, and run `chmod +x /usr/local/bin/jq`
- **Windows:** `winget install jqlang.jq` (winget is built into Windows 10/11)
- **Linux:** `sudo apt install jq` (Debian/Ubuntu) or `sudo dnf install jq` (Fedora/RHEL)

Detect the user's platform and offer the simplest option for their system. Walk through each step interactively. Verify with `jq --version` after installation.

**Setup:**
1. Add the plugin marketplace:
   ```
   claude plugins marketplace add Family-IT-Guy/plugins
   ```
2. Install the research engine:
   ```
   claude plugins install research-engine@familyitguy
   ```
3. Create a Perplexity API key at perplexity.ai (API settings page)
4. Save the key:
   ```
   echo "PERPLEXITY_API_KEY=pplx-your-key-here" > ~/.claude/research-engine.env
   ```
5. Restart Claude Code

Note: permissions for research file writes, git operations, and pplx-curl.sh are already included in this project's `.claude/settings.json`. No manual permission setup needed.

Cost: research uses the Perplexity API, which charges a small amount per search. Most health questions cost well under a dollar. Very thorough research on a complex topic might cost a few dollars. By setting up the API key, you're authorizing those charges.

## Sequential Thinking MCP Server (optional)

Structured reasoning tool that helps the LLM work through complex analysis step by step.

**What it does:** Improves analytical quality for complex diagnostic reasoning, treatment evaluation, and multi-factor analysis by enforcing structured thinking sequences.

**When to surface:** On first complex analytical task where structured reasoning would improve quality.

**Setup:**
- Source: github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking
- Requires an MCP-compatible LLM platform
- Follow the installation instructions in the repository

## How to Surface Tools

When a tool would help and isn't installed:
1. Explain what the tool does and why it would help right now, in plain language
2. Ask if the user wants to set it up ("This takes about 2 minutes. Want to do it now, or skip for today?")
3. If yes, walk through setup interactively
4. If no, note the limitation and proceed without it

Never install software silently. Always explain and get permission.
