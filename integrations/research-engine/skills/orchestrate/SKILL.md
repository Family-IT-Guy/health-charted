---
name: research-engine:orchestrate
description: >-
  Deep web research via Perplexity Sonar API with cited sources, reasoning
  traces, and persistent research threads. ALWAYS use instead of WebSearch,
  WebFetch, and firecrawl for: (1) Any research request ("research this",
  "look into", "find out about", "what's the latest on"), (2) Current events,
  news, recent developments, (3) Fact-checking or verifying claims,
  (4) Cross-library comparisons, architecture decisions, technology landscape
  research (use context7 for single-library docs only), (5) Market research,
  competitive analysis, business intelligence, (6) Troubleshooting, debugging,
  root cause analysis, (7) Any question needing web-grounded cited data.
  Triggers: "research", "look up", "search for", "investigate", "deep dive",
  "what do we know about", "dig into". Extracts topics from voice memos,
  emails, and notes. Also called by the daily brief during Phase 6.75.
  Dispatches background sub-agents for parallel research execution.
---

# Research Engine: Orchestrate

Plugin root: `integrations/research-engine/`
All `references/` paths below are relative to this root.

Scan inputs for researchable topics, check existing coverage, get user approval,
and dispatch background sub-agents. This is the only user-facing research
dispatch skill. The :execute skill is internal and should never be invoked
directly.

Two entry paths:
1. **On-demand**: user provides a specific topic, voice memo, email, or note
2. **Daily brief (Phase 6.75)**: receives a pre-collected research queue from
   triage -- skip extraction, start at coverage check

## Workflow

### Step 1: Determine Input Source

**If called with a specific note/email/topic:**

Read the full content of the input. Record:
- `source_type`: one of `voice-memo`, `email`, `note`, `manual`
- `source_ref`: filename, email ID, or topic string

**If called by daily brief (Phase 6.75):**

Receive the collected research queue items (flagged during triage +
system-identified topics). Each item already has `source_type` and `source_ref`
from the brief's triage. Skip Step 2 and proceed to Step 3.

### Step 2: Extract Researchable Topics

Apply LLM judgment to the input text. Use the topic extraction checklist in
`references/orchestration.md` as a light nudge (not an exhaustive filter).

For each identified topic, generate:

| Field | Description |
|-------|-------------|
| topic_name | Concise, 3-8 words |
| why_researchable | One sentence: what knowledge gap exists |
| research_query | Specific, actionable query (sharp enough for Perplexity) |
| source_reference | Which line/section of the input it came from |
| recommended_depth | `quick` or `deep` (see depth tiers in `references/orchestration.md`) |

Not every mention warrants research. Apply judgment: is the user likely to act
on this information, or is it passing context? When in doubt, extract it -- the
user decides what to keep in Step 4.

### Step 3: Check Existing Coverage

For each extracted topic, run the 4-system parallel search from
`references/coverage-check.md` inline (skills cannot invoke other skills).

The return structure:

```yaml
topic: "<topic string>"
coverage: full | partial | none | stale
research_ids: [...]
hit_count: <N>
hits_by_system:
  research_files: <N>
```

Filter decisions:
- **full**: drop (already researched)
- **stale**: keep, note it is a refresh
- **partial** or **none**: keep
- **upgrade candidate**: If coverage is `full`, check the depth of existing
  threads: read the YAML frontmatter of each file listed in `research_ids`
  (resolve paths via the instruction below). If ALL matched thread files have
  `depth: quick` (none has `depth: deep`), and the user's recommended or
  requested depth is `deep`, flag the topic as an upgrade candidate. Keep it
  in the list with a note: `upgrade_from: RE-YYYY-MMDD-NNN` (the existing
  quick scan's research_id). Record the matched thread file path as
  `upgrade_thread_path` for use in Step 5.

  **To resolve a research_id to a file path:** Glob `./research/*-*.md`
  and read the first 5 lines of each result to find the file whose frontmatter
  `research_id` field matches. Alternatively, the `:retrieve` quick-status
  response includes filenames in its `research_files` hits — use those paths
  directly if available.

If no topics remain after filtering, report: "All extracted topics already have
full research coverage. Nothing to dispatch." Then stop.

### Step 4: Present Research Plan to User

Present a scannable summary of all topics, with detail available on demand.

**Summary view** (always show this first):

```
RESEARCH PLAN (N topics from [source_type]: "[source_ref]"):

1. [Topic name] — [none / partial / stale (N days)]
   → [recommended depth]: "[research query]"

2. [Topic name] — [none / partial / stale (N days)]
   → [recommended depth]: "[research query]"

3. [Topic name] — quick scan (RE-YYYY-MMDD-NNN, YYYY-MM-DD) → upgrade to deep
   → [recommended depth]: "[research query]"

Detail [1/2/3/all]? Or approve with depths:
e.g., "1: quick, 2: skip, 3: deep" or "quick on all"
```

For upgrade candidates, append `→ upgrade to deep` on the coverage line.

**Expanded view** (show when user says "detail N" or "detail all"):

```
N. [Topic name]
   Objective: [What this research will answer]
   Query: "[the specific research query]"
   Methodology: [sonar-reasoning-pro / sonar-deep-research] because [rationale]
   Search mode: [web/academic/sec] because [rationale]

   Parameters:
   - return_images: [true/false]
   - Date filters: [suggest if applicable, omit if not]
   - Domain filters: [suggest if applicable, omit if not]

   Alternatives:
   - [Alternative framing] — might be better if [condition]

   Blind spots:
   - [What this approach might miss]

   → quick / deep / skip?
```

For upgrade candidates in expanded view, add:
```
   Prior research: quick scan (RE-YYYY-MMDD-NNN, YYYY-MM-DD)
   → deep (upgrade) / deep (fresh) / skip?
```

"deep (upgrade)" passes the quick scan's findings as context.
"deep (fresh)" dispatches a standard deep dive ignoring prior work.

**Handling user responses:**

Accept formats like "1: deep, 2: quick, 3: skip", "quick on all", or one at
a time. Users may also edit queries, parameters, or scope before approving.
If the user approves without requesting detail, proceed with the recommended
depths shown in the summary view.

### Step 5: Generate Research IDs and Dispatch Sub-Agents

**Generate all research IDs before dispatching any sub-agents** to avoid race
conditions. Use the ID generation script in `references/orchestration.md`
(format: `RE-YYYY-MMDD-NNN`, sequential based on existing thread files for
today's date).

**Per-topic dispatch routing:**
For each approved topic, use exactly ONE of these paths:
- Standard topic (not an upgrade candidate): use the standard dispatch template below
- User chose "deep (upgrade)": use the **upgrade dispatch procedure** below
- User chose "deep (fresh)": use the standard dispatch template with `depth: deep`
  (ignore prior quick scan entirely)

For each approved topic, dispatch a background sub-agent using the Agent tool
with `run_in_background: true` and `subagent_type: "research-executor"`. Use
the sub-agent dispatch template from `references/orchestration.md`, filling in
all variables:

- `query`: the specific research query from Step 2
- `depth`: user's chosen depth from Step 4
- `research_id`: pre-generated ID
- `source_type`: from Step 1 (for cascade children, inherit from original parent)
- `source_ref`: from Step 1 (for cascade children, inherit from original parent)
- `triggered_by`: `daily-brief` or `on-demand`

The `research-executor` agent has `permissionMode: acceptEdits` and explicit
tool declarations (Read, Write, Bash, Glob, Grep, WebFetch, WebSearch). Its
system prompt instructs it to read and follow the :execute skill file.

**For upgrade dispatches** (user chose "deep (upgrade)"):

1. Read the existing quick scan thread file (path from Step 3)
2. Extract from the thread file:
   - Key findings (from the "### Findings" section)
   - Sources already consulted (from the "### Sources" section)
   - Related questions (from "### Related Questions" if present, for breadth)
3. Construct `context_from_parent`:
   ```
   KEY FINDINGS FROM QUICK SCAN ({quick_scan_research_id}):
   {extracted findings}
   SOURCES ALREADY CONSULTED:
   {source list}
   BUILD ON THESE — do not re-discover what's above.
   INVESTIGATE DEEPER:
   {original query, refined for depth}
   ```
   For `{source list}`, extract from `### Sources`: include URL and title only,
   one per line. Example: `https://example.com — Article Title`. Do not include
   full citation metadata — just enough for the deep dive to avoid re-fetching
   the same sources.
4. Set `cascade_chain` with the quick scan as the first entry:
   ```yaml
   cascade_chain:
     - id: {quick_scan_research_id}
       topic: "{topic_slug}"
       key_finding: "{one-line summary of quick scan}"
   ```
5. Dispatch using the **cascade child dispatch template** from
   `references/orchestration.md`. The new deep dive gets its own research_id.
6. After completion, the deep dive's thread file will have
   `cascade_parent: {quick_scan_research_id}` in its frontmatter,
   making the lineage traceable.

Report to user: "Dispatched N research sub-agents. They will run in the
background."

### Step 5.5: Report Completion Stats

When all dispatched sub-agents have completed (or failed), present aggregate
stats to the user:

```
RESEARCH DISPATCH COMPLETE:
  Agents: N succeeded, M failed
  Total cost: $X.XX
  Total sources: N
  Thread files: [list of paths]
  Cascades requested: N
```

**If any agent used WebSearch fallback**, add a prominent warning:

```
⚠️ FALLBACK USED:
  RE-YYYY-MMDD-NNN: [topic] — [fallback_reason]
  Perplexity attempts: N | Retry params: [what was changed]
  → Want me to retry this topic with Perplexity? (suggest adjusted parameters)
```

Do NOT bury fallback usage in the thread file metadata. Surface it here so
the user can decide whether the degraded results are acceptable or worth
retrying.

Sum `cost` and `source_count` from each sub-agent's structured return summary.
For failed agents, note the failure reason. If any agents requested cascades,
proceed to Step 6.

### Step 6: Handle Cascade Returns

When a background sub-agent completes, check whether it wrote a cascade file
at `research/cascades/[research_id]-cascade.yml`.

If no cascade file exists, the research is complete. Report the sub-agent's
summary to the user.

If a cascade file exists:

1. Read the cascade YAML file
2. **Handle blocked sources** (if `blocked_sources` section exists):
   For each blocked URL, attempt to fetch the content using main-agent tools
   (which have full permissions unlike sub-agents):
   - First try **chawan-web-fetch** skill (terminal browser, handles light JS)
   - If chawan fails, try **playwright-cli** skill (full browser, handles
     CAPTCHAs, session cookies, JavaScript-heavy sites)
   - If content is retrieved, save to `research/sources/` and append
     the key findings to the cascade's `context_for_next_agent` fields where
     relevant. This gives cascade children the data the parent couldn't access.
   - If all methods fail, note it when presenting follow-ups so the user knows.
3. Present follow-up topics using the same compact format as Step 4:

**Summary view:**

```
CASCADE from [parent_topic] ([parent_research_id]):
[parent_summary — 2-3 sentences]

Blocked sources: [N retrieved / M failed] (or omit if none)

Follow-up topics:
1. [Topic name] — from parent gap
   → [recommended depth]: "[suggested query]"

2. [Topic name] — from parent gap
   → [recommended depth]: "[suggested query]"

Detail [1/2/all]? Or approve with depths:
e.g., "1: quick, 2: skip" or "quick on both"
```

**Expanded view** (when user says "detail N"):

```
N. [Topic name]
   Why: [why this follow-up is needed]
   Query: "[suggested query]"
   Context from parent: [1-line summary of what parent found]

   → quick / deep / skip?
```

4. If approved, generate new research IDs and dispatch sub-agents using the
   **cascade child dispatch template** from `references/orchestration.md`

When building cascade child parameters:
- Copy `cascade_chain` from the parent's cascade YAML as-is (the writing agent
  already includes itself as the last entry -- do NOT append the parent again)
- Copy `context_for_next_agent` from the cascade YAML into the
  `context_from_parent` parameter. If step 2 retrieved blocked source content,
  append it to the relevant `context_from_parent` fields.
- Inherit `source_type` and `source_ref` from the original parent (trace back
  to the voice memo/email that started the chain)

5. Continue until no more cascades or user says stop

## Error Handling
- If sub-agent dispatch fails, report the failure and the topic to the user.
  Offer to retry.
- If a sub-agent returns an error instead of a summary, surface it with the
  topic name and research_id. Do not auto-retry API key or auth errors.

## Daily Brief Integration

When called from the daily brief's Phase 6.75, the input is a pre-collected
list of research opportunities rather than raw text. Each item includes
`source_type`, `source_ref`, a topic name, and a suggested query.

Workflow adjustment:
- **Skip Step 2** (extraction already done during triage)
- **Start at Step 3** (check existing coverage) with the provided topics
- Steps 4-6 proceed normally

## References

Detailed templates and definitions live in the reference files. Do not duplicate
here.

- `references/orchestration.md` -- Topic extraction checklist, depth tier
  definitions, cascade rules, research ID generation script, sub-agent dispatch
  templates (standard + cascade child), variable reference table
- `references/models.md` -- Model capabilities and pricing
