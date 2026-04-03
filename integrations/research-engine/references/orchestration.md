# Orchestration Reference

## Topic Extraction Checklist

When scanning input (voice memos, emails, notes), look for these researchable
signals. This is a light nudge, not an exhaustive taxonomy -- one or two hits is
enough to justify a research dispatch.

- **People or organizations** mentioned that could be investigated
- **Claims or assertions** that could be validated with evidence
- **Decisions** that would benefit from comparison data or alternatives analysis
- **Ideas or strategies** that could benefit from prior art research
- **Tools, products, or services** worth evaluating
- **Unfamiliar concepts or terms** worth defining
- **YouTube URLs** shared by the user or found in research — ingest transcript as primary source
- **Topics where video content exists** (tutorials, expert talks, interviews) — search YouTube for relevant videos

Not every mention warrants research. Apply judgment: is the user likely to act on
this information, or is it passing context? When in doubt, extract it -- the
orchestrator presents candidates for user approval before dispatching.

## YouTube Transcript Ingestion

YouTube videos are a primary source type. When a research topic would benefit
from video content (expert talks, tutorials, interviews, competitor analysis),
YouTube transcripts can be ingested during deep dives.

**During deep dives:** Sub-agents can ingest YouTube transcripts as primary
sources using yt-dlp (must be installed: `pip install yt-dlp` or `brew install yt-dlp`):

```bash
yt-dlp --dump-json --skip-download "VIDEO_URL"
```

Returns JSON with `title`, `channel`, `duration_string`, `view_count`,
and more. Extract the transcript and save to
`research/sources/TIMESTAMP_youtube-VIDEO_ID.md` alongside other
primary sources.

**When to ingest:** High view count + authoritative channel + directly
relevant to the research query. Don't ingest every video found. 1-3 per
research topic is the practical limit (each transcript is ~2,500 words per
15 minutes of video).

**YouTube search for topic discovery:**
```bash
yt-dlp --dump-json --flat-playlist --skip-download "ytsearch5:QUERY"
```

## Depth Tier Definitions

### Quick Scan

Single sonar-reasoning-pro call. Perplexity synthesis only -- no primary source
fetching, no cascade assessment.

- **Model**: sonar-reasoning-pro
- **Primary sources**: No
- **Cascade assessment**: No
- **Typical cost**: ~$0.01-0.05
- **Use when**: Factual lookups, definitions, simple comparisons, current events,
  anything where Perplexity's synthesis is sufficient on its own

### Deep Dive

May use sonar-deep-research or multi-model synthesis. Fetches and reads primary
sources (wave pattern). Assesses completeness and writes cascade requests when
gaps remain.

- **Model**: sonar-deep-research or sonar-reasoning-pro (per query complexity)
- **Primary sources**: Yes (fetched and read by sub-agents)
- **Cascade assessment**: Yes (writes cascade YAML if critical gaps found)
- **Typical cost**: ~$0.10-0.50
- **Use when**: High-stakes decisions, due diligence, topics requiring source
  verification, anything where Perplexity's synthesis alone is insufficient

## Cascade Rules

### When to Write a Cascade Request

A sub-agent writes a cascade request (`research/cascades/[research_id]-cascade.yml`)
when it identifies a **specific gap** that requires focused follow-up research.
Cascades are deep-dive only -- quick scans never cascade.

### Good Cascades

- **Specific**: targets a narrow, well-defined sub-question
- **Builds on parent**: uses parent findings as a foundation, not a fresh start
- **Addresses a real gap**: the parent research could not answer this question
  and the answer matters for the user's needs
- **Actionable**: the follow-up query is concrete enough to yield useful results

### Bad Cascades

- **Too broad**: "research everything about X" (should be its own top-level topic)
- **Duplicates parent**: re-asks what the parent already answered
- **Tangential**: interesting but not relevant to the original research goal
- **Minor gap**: a small uncertainty that does not block understanding

### cascade_chain Format

The `cascade_chain` array traces the full ancestry of a cascade child. Each entry
captures the research ID, topic, and the key finding that motivated the next step.
The orchestrator builds this from the parent's cascade request file and passes it
to the child sub-agent.

```yaml
cascade_chain:
  - id: RE-2026-0302-001
    topic: "multi-agent research systems"
    key_finding: "Three frameworks identified, delegation unclear"
  - id: RE-2026-0302-003
    topic: "LangGraph vs CrewAI delegation"
    key_finding: "LangGraph supports multi-level, CrewAI limited to one level"
```

When dispatching a cascade child, the orchestrator:

1. Reads the parent's cascade YAML file
2. Copies the `cascade_chain` array as-is (the writing agent already includes
   itself as the last entry — do NOT append the parent again)
3. Passes the chain to the child sub-agent

## Research ID Format

Format: `RE-YYYY-MMDD-NNN`

- `YYYY`: four-digit year
- `MMDD`: two-digit month, two-digit day (no separator)
- `NNN`: zero-padded sequential number, starting at 001

Examples: `RE-2026-0302-001`, `RE-2026-0302-012`

### Generating the Next ID

The orchestrator generates all research IDs before dispatching sub-agents. To
determine the next NNN, count existing thread files for today's date (each
research dispatch creates one `*-YYYY-MM-DD.md` thread file):

```bash
TODAY_ID=$(date +%Y-%m%d)       # 2026-0302 (ID format, no MM-DD hyphen)
TODAY_FILE=$(date +%Y-%m-%d)    # 2026-03-02 (file suffix format)
EXISTING=$(ls ./research/*-${TODAY_FILE}.md 2>/dev/null | wc -l | tr -d ' ')
NEXT=$(printf "%03d" $((EXISTING + 1)))
RESEARCH_ID="RE-${TODAY_ID}-${NEXT}"
echo "$RESEARCH_ID"
```

When dispatching multiple sub-agents in parallel, pre-generate all IDs before
dispatch (e.g., 001, 002, 003) to avoid race conditions.

## Sub-Agent Dispatch Template

This is the exact prompt structure the orchestrator passes to the Agent tool when
dispatching a sub-agent to run :execute. Variables in `{{BRACES}}` are filled in
by the orchestrator.

**IMPORTANT**: All dispatches MUST use `subagent_type: "research-executor"` (not
`"general-purpose"`). The research-executor agent definition at
`integrations/research-engine/agents/research-executor.md` declares the required tools and sets
`permissionMode: acceptEdits` to ensure Write access in background execution.

### Standard Dispatch (No Cascade)

```
You are a research sub-agent. Follow the research-engine:execute skill exactly.

## Parameters

- query: "{{QUERY}}"
- depth: "{{DEPTH}}"
- research_id: "{{RESEARCH_ID}}"
- source_type: "{{SOURCE_TYPE}}"
- source_ref: "{{SOURCE_REF}}"
- triggered_by: "{{TRIGGERED_BY}}"

## Instructions

1. Read the skill at integrations/research-engine/skills/execute/SKILL.md
2. Execute the {{DEPTH}} scan workflow for the query above
3. Return your structured summary when complete

Do NOT present a research plan or wait for approval. Execute the research immediately.
```

### Cascade Child Dispatch

```
You are a research sub-agent. Follow the research-engine:execute skill exactly.

## Parameters

- query: "{{QUERY}}"
- depth: "{{DEPTH}}"
- research_id: "{{RESEARCH_ID}}"
- source_type: "{{SOURCE_TYPE}}"
- source_ref: "{{SOURCE_REF}}"
- triggered_by: "{{TRIGGERED_BY}}"
- cascade_chain:
{{CASCADE_CHAIN_YAML}}
- context_from_parent: |
{{CONTEXT_FROM_PARENT}}

## Instructions

1. Read the skill at integrations/research-engine/skills/execute/SKILL.md
2. This is a cascade child — incorporate context_from_parent into your research
3. Execute the {{DEPTH}} scan workflow for the query above
4. Return your structured summary when complete

Do NOT present a research plan or wait for approval. Execute the research immediately.
```

### Variable Reference

| Variable | Source | Example |
|----------|--------|---------|
| `{{QUERY}}` | Extracted topic or cascade follow-up query | `"LangGraph vs CrewAI cascading delegation comparison 2026"` |
| `{{DEPTH}}` | Orchestrator decision or cascade `recommended_depth` | `"quick"` or `"deep"` |
| `{{RESEARCH_ID}}` | Pre-generated by orchestrator (see ID format above) | `"RE-2026-0302-001"` |
| `{{SOURCE_TYPE}}` | Origin of the input that triggered research. For cascade children, inherit from the original parent (traces back to the voice memo/email that started the chain). | `"voice-memo"`, `"email"`, `"note"`, `"manual"` |
| `{{SOURCE_REF}}` | Identifier for the source material. For cascade children, inherit from the original parent. | `"Multi-Agent Research Framework.md"` |
| `{{TRIGGERED_BY}}` | What initiated the research session | `"daily-brief"` or `"on-demand"` |
| `{{CASCADE_CHAIN_YAML}}` | YAML array from parent cascade file, indented 2 spaces (aligns with `cascade_chain:` key) | See cascade_chain format above |
| `{{CONTEXT_FROM_PARENT}}` | `context_for_next_agent` field from parent cascade YAML, indented 4 spaces (block scalar content must exceed the `context_from_parent: \|` key indent) | See :execute SKILL.md Step 8 |
