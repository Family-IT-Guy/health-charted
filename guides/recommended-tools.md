# Recommended Tools

Tools that enhance the system. None are required to start using Health Charted, but each improves the quality of specific capabilities. These are surfaced to the user on first need, not as a prerequisite.

## Perplexity API (strongly recommended)

Powers the research engine for structured, cited medical research with academic and web search modes.

**What it does:** Enables deep, citeable research with source verification. Without it, research falls back to whatever the LLM platform provides natively, which is less comprehensive and harder to trace to primary sources.

**When to surface:** On first research need. If the user asks to research a health topic and `integrations/research-engine/.env` has no API key configured, pause and offer setup.

**Setup:**
1. Create an account at perplexity.ai
2. Go to API settings and generate an API key
3. Create the file `integrations/research-engine/.env` with the content: `PERPLEXITY_API_KEY=your-key-here`
4. Cost: pay-per-query. A typical research session costs a few dollars.

## yt-dlp (optional)

Downloads and extracts audio and transcripts from video content.

**What it does:** When research leads to video sources (conference talks, medical lectures, expert interviews), yt-dlp can download the content for analysis.

**When to surface:** On first need to process a video URL during research.

**Setup:**
- Mac: `brew install yt-dlp`
- Any platform: `pip install yt-dlp`
- Or download directly from github.com/yt-dlp/yt-dlp

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
3. If yes, walk through setup interactively, running commands with the user's permission
4. If no, proceed with the best available alternative and note the limitation

Never install software silently. Always explain and get permission.
