# Recommended Tools

Tools that enhance the system. None are required to start using Health Charted, but each improves the quality of specific capabilities. These are surfaced to the user on first need, not as a prerequisite.

## Research Engine Plugin (strongly recommended)

Cited web research via the Perplexity Sonar API. Background sub-agents, primary source verification, cascade follow-ups.

**What it does:** Enables deep, citeable research with source verification and evidence grading. Without it, research is limited to whatever the LLM platform provides natively.

**When to surface:** On first research need. If the user asks to research a health topic and the research-engine plugin is not installed, pause and offer setup.

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
5. Add permissions to `~/.claude/settings.json` under `permissions.allow`:
   ```json
   "Edit(/research/**)",
   "Bash(*pplx-curl.sh*)",
   "Bash(*yt-dlp*)"
   ```
6. Restart Claude Code

Cost: pay-per-query. A typical research session costs a few dollars.

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
