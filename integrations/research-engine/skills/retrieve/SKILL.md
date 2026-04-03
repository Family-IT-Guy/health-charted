---
name: research-engine:retrieve
description: >-
  Knowledge retrieval from research files. Invoke when the user asks "what do we
  know about X?", "have we researched X?", "any notes on X?", or before
  dispatching new research to check existing coverage.
  Also used by :orchestrate for batch coverage checks.
---

# Research Engine: Retrieve

Search research files, present results, assess coverage, and offer to fill
gaps via :orchestrate.

## Input

| Parameter | Required | Description |
|-----------|----------|-------------|
| topic | yes | The query string — what the user wants to know about |
| mode | no | `"interactive"` (default) or `"quick-status"` (structured data, no presentation) |

## Workflow

### Step 1: Search Research Files

Search for existing research on the topic:

```
./research/*.md
```

Use Glob to find candidate files by name, then Grep to search content for
keyword relevance. For matches, Read the first 20 lines to extract YAML
frontmatter (research_id, depth, date, source_type).

### Step 2: Assess and Report

**Skip this step if mode is `quick-status`** — go to Quick-Status Mode below.

Present results:

```
FROM RESEARCH FILES:
  [research_id]: "[topic]" (depth, date)

NO RESEARCH FOUND ON: [topic]
```

Assess overall coverage:

| Coverage | Criteria |
|----------|----------|
| **Full** | Dedicated research thread exists with deep dive |
| **Partial** | Research exists but doesn't fully cover the topic |
| **None** | No hits |
| **Stale** | Research exists but older than 30 days |

### Step 3: On-Demand Deep Read

Only if the user asks to read a specific hit. Dispatch a sub-agent to:

1. Read the full content of the selected file
2. Summarize it in the context of the user's current question
3. Highlight what it covers and what gaps remain

### Step 4: Handoff to :orchestrate

If coverage is **none** or **partial**, offer:

> "Want me to research this topic? I can kick off research-engine:orchestrate."

If coverage is **stale**, offer:

> "Prior research is from [date]. Want me to refresh it?"

Do not auto-dispatch — wait for user confirmation.

## Quick-Status Mode

When called by :orchestrate (mode = `quick-status`), skip presentation and
return structured data only:

```yaml
topic: "<topic string>"
coverage: full | partial | none | stale
research_ids: ["RE-2026-0302-001"]  # empty array if none
hit_count: <total hits>
```

This allows batch processing — :orchestrate can check coverage for multiple
topics in rapid succession without verbose output.

## Error Handling

- **Research files**: Directories don't exist → report 0 hits, continue
