---
name: research-engine:query
description: >-
  Inline single-topic research via Perplexity Sonar API. Use for quick lookups,
  fact-checking, and single questions that don't need multi-agent orchestration.
  Runs in the main conversation — no background sub-agents. Presents a research
  plan for approval before executing. Triggers: "quick research", "look up",
  "what is", "check if", "verify", "is it true that". For multi-topic research,
  voice memo scanning, or deep cascading research, use :orchestrate instead.
---

# Research Engine: Query

Plugin root: `integrations/research-engine/`
All `references/` paths below are relative to this root.

Lightweight inline research. Runs Perplexity directly in the main agent context.
No sub-agents, no coverage check, no cascade assessment, no primary source
reading. Presents results in the conversation immediately.

## Workflow

### Step 1: Plan the Research

Before any API call, determine parameters using LLM reasoning (NO tool calls):

1. **topic_slug**: Lowercase, hyphens, no spaces (e.g., "claude-code-hooks-api")
2. **model**: Binary decision per `references/models.md`:
   - Exhaustive, report-style research? -> `sonar-deep-research`
   - Everything else -> `sonar-reasoning-pro` (default)
3. **system_prompt**: Based on query context:
   - **Software/technical**: "Prioritize official documentation, GitHub repositories, and high-quality technical sources. Include code examples when relevant. Note version compatibility."
   - **Academic/scientific**: "Prioritize peer-reviewed sources and reputable publications. Note methodology limitations. Highlight conflicting findings."
   - **Fact-checking**: "Cross-reference claims against multiple independent sources. Distinguish verified facts from disputed claims. Rate confidence levels."
   - **Business/market**: "Prioritize authoritative sources (SEC filings, official reports). Include quantitative data. Note potential biases."
   - **RCA/debugging**: Use the RCA system prompt from `references/rca-workflow.md`.
4. **search_mode** (auto-select):
   - Studies, papers, peer-reviewed, scientific -> `"academic"`
   - Companies, filings, earnings, SEC, financial -> `"sec"`
   - Everything else -> `"web"`
5. **return_images** (auto-select):
   - Product research, visual topics, places, design -> `true`
   - Technical, analytical, factual -> `false`
6. **date_filters** (auto-suggest):
   - Query contains temporal signals ("recent", "latest", "2025", "last month",
     "since January") -> suggest `search_after_date_filter` set to 6 months ago
   - No temporal signal -> omit
7. **domain_filters** (auto-suggest):
   - Query targets specific ecosystem (Python -> `docs.python.org, github.com`;
     legal -> `.gov` domains; Apple -> `developer.apple.com`) -> suggest filters
   - General query -> omit

For all available API parameters: see `references/api-reference.md`.

### Step 2: Present Research Plan

**MANDATORY**: Present the plan and await explicit user approval.

```
## Research Plan

**Objective**: [Restate the specific question in precise terms]
**Scope**: [What's in] | [What's explicitly out]
**Methodology**: [Model selected] because [rationale]
**Search mode**: [web/academic/sec] because [rationale]

### Parameters
- search_mode: [web/academic/sec]
- return_images: [true/false]
- Date filters: [suggest if applicable, e.g., "search_after_date_filter: 09/05/2025"]
- Domain filters: [suggest if applicable, e.g., "search_domain_filter: ['docs.python.org']"]

### Alternatives to Consider
- [Alternative framing 1] — might be better if [condition]
- [Alternative framing 2] — worth considering because [reason]

### Potential Blind Spots
- [What this approach might miss]
- [Bias in source prioritization]

Approve / edit / skip?
```

**Do not execute the API call until user approves or provides direction.**

If the user edits the plan, incorporate their changes. If "skip", abort.

### Step 3: Execute API Call

Single Bash call to pplx-curl.sh. This is the ONLY Bash command in the workflow.

```bash
integrations/research-engine/scripts/pplx-curl.sh --research "TOPIC_SLUG" '{"model":"MODEL_NAME","messages":[{"role":"system","content":"SYSTEM_PROMPT"},{"role":"user","content":"USER_QUERY"}],"web_search_options":{"search_context_size":"high"},"search_mode":"SEARCH_MODE","return_related_questions":true,"return_images":RETURN_IMAGES,"temperature":0.2,"stream":false}'
```

The `--research` mode handles internally:
- Creating `research/raw/`, `sources/`, `cascades/` directories
- Generating a timestamp for the filename
- Saving the API response to `research/raw/TIMESTAMP_TOPIC-SLUG.json`
- Printing the output file path as `OUTPUT_PATH=<path>` (last line of output)

Parse the last line of output to get the file path for Step 4.

**For deep research, add to the JSON body:**
```json
"reasoning_effort": "high"
```

**With date filters (when approved in research plan):**
```json
"search_after_date_filter": "01/01/2025",
"search_before_date_filter": "03/05/2026"
```

**With domain filters (when approved in research plan):**
```json
"search_domain_filter": ["docs.python.org", "github.com"]
```

**CRITICAL**: `search_context_size` MUST be nested inside `web_search_options`.
Top-level placement is silently ignored (falls back to "low"). This is the
single most common API mistake.

### Step 4: Read and Extract

Use the **Read** tool to retrieve the saved JSON file. For large responses
(>2000 lines), use offset/limit to read in chunks.

Extract from the response:
- Content: `.choices[0].message.content`
- Sources: `.search_results[]` (title, URL, date, snippet) -- use this, not `citations`
- Cost: `.usage.cost.total_cost`
- Token count: `.usage.total_tokens`
- Related questions: `.related_questions` (if present)

### Step 5: Write Thread File

Use the **Write** tool to create `research/[topic-slug]-YYYY-MM-DD.md`
with YAML frontmatter.

**YAML Frontmatter**:

```yaml
---
research_id: RE-2026-0305-001
source_type: manual
source_ref: "inline query"
depth: quick
triggered_by: inline
date: 2026-03-05
queries: 1
models_used: [sonar-reasoning-pro]
cost_estimate: $0.02
cascade_parent: null
---
```

**Thread body format**:

```markdown
# [Topic Name]

## Approved Plan
**Objective**: [from approved plan]
**Scope**: [from approved plan]
**Methodology**: [from approved plan]

## Query 1: [Brief query description]
**Timestamp**: YYYY-MM-DDTHH:MM:SSZ
**Raw**: [raw/TIMESTAMP_topic-slug.json](raw/TIMESTAMP_topic-slug.json)
**Model**: sonar-reasoning-pro | **Tokens**: N | **Cost**: $X.XX

### Findings
[Synthesized findings -- key points, not full dump]

### Sources
[1] Title - domain.com
    URL | Published: YYYY-MM-DD
[2] Title - domain.com
    URL | Published: YYYY-MM-DD

### Related Questions
- [From API response, if present]

## Synthesis (Updated: YYYY-MM-DD)

### Key Conclusions
- [Conclusion 1]
- [Conclusion 2]

### Open Questions
- [What remains unresolved]

### Confidence Assessment
- High confidence: [topics]
- Moderate confidence: [topics]
- Needs verification: [topics]
```

**Research ID generation**: Use the same ID format as :execute
(`RE-YYYY-MMDD-NNN`). To get the next sequential number:

```bash
TODAY_ID=$(date +%Y-%m%d)
TODAY_FILE=$(date +%Y-%m-%d)
EXISTING=$(ls ./research/*-${TODAY_FILE}.md 2>/dev/null | wc -l | tr -d ' ')
NEXT=$(printf "%03d" $((EXISTING + 1)))
RESEARCH_ID="RE-${TODAY_ID}-${NEXT}"
echo "$RESEARCH_ID"
```

Run this BEFORE the pplx-curl.sh call (Step 3) so the ID is available for
the thread file. This is the one exception to the "single Bash call" rule —
ID generation is a separate Bash call.

### Step 6: Present Results Inline

Present findings directly in the conversation:

```
## Research Results: [Topic Name]

[Synthesized findings — key points organized by relevance]

### Sources
[1] Title — domain.com
    URL | Published: YYYY-MM-DD
[2] ...

**Cost**: $X.XX | **Model**: sonar-reasoning-pro | **Tokens**: N
**Thread file**: research/[topic-slug]-YYYY-MM-DD.md

### Related Questions
- [If present — user may want to follow up]
```

## Error Handling

### HTTP Errors (non-zero exit from pplx-curl.sh)

- HTTP 429 (rate limited): Wait 10 seconds, retry once
- HTTP 500 (server error): Retry once after 5 seconds
- HTTP 401 (unauthorized): Abort — API key issue, inform user
- HTTP 400 (bad request): Attempt to reformulate (remove special characters,
  shorten query), retry once
- All other failures after retry: Abort and report error to user

### Irrelevant Results (Perplexity succeeded but content is off-topic)

If the API call returns but content doesn't address the query:

1. **Diagnose**: date filters too narrow? domain filters excluding results?
   query too specific? search_mode mismatch?
2. **Retry with adjusted parameters** — present the adjustment to the user:
   "Perplexity returned irrelevant results. Retrying with [change]. Approve?"
3. If retry also fails, offer the user a choice:
   - Adjust parameters further
   - Fall back to WebSearch/WebFetch (flag quality reduction)
   - Abort

### Fallback to WebSearch/WebFetch

Use ONLY when pplx-curl.sh is inaccessible (Bash denied) or Perplexity failed
after retry AND user approved fallback.

**When falling back:**
1. Flag prominently in results presentation:
   ```
   ⚠️ FALLBACK: Results from WebSearch, not Perplexity
   Reason: [why Perplexity failed]
   Offer: "Want me to retry Perplexity with different parameters?"
   ```
2. Set `models_used: [web-search-fallback]` in thread file
3. Add `fallback_reason` field to YAML frontmatter

## References

- `references/api-reference.md` -- Full API docs, parameters, response format
- `references/models.md` -- Model capabilities, pricing
- `references/rca-workflow.md` -- Root cause analysis workflow
