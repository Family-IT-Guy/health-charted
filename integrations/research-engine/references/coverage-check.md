# Coverage Check: Research File Search

Search research files for existing coverage on a topic. Used by :orchestrate
(which cannot invoke :retrieve directly).

## Research Files

```
./research/*.md
```

Use Glob to find candidate files by name, then Grep to search content for
keyword relevance. For matches, Read the first 20 lines to extract YAML
frontmatter (research_id, depth, date, source_type).

## Error Handling

- **Research files**: Directories don't exist → report 0 hits
