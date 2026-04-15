---
type: schema
updated: 2026-04-15
---

# Wiki Schema

Conventions the LLM follows when maintaining this wiki. Co-evolve this file with the user over time.

## Three layers

1. **Raw sources** (`raw/`) — immutable. LLM reads but never modifies. Articles, papers, PDFs, clippings. Images go in `raw/assets/`.
2. **Wiki** (everything else) — LLM-owned. Summaries, entity pages, concept pages, syntheses. Cross-linked with Obsidian wikilinks.
3. **Schema** (this file + `./.claude/`) — how the wiki is structured and how the LLM behaves.

## Directory layout

```
raw/          sources as originally captured (immutable)
  assets/     downloaded images
summaries/    one summary page per raw source (type: summary)
entities/     people, orgs, places, products (type: entity)
concepts/     ideas, topics, techniques (type: concept)
comparisons/  X vs Y, tradeoff analyses (type: comparison)
overview/     bird's-eye maps of a domain or the whole wiki (type: overview)
syntheses/    multi-source analyses, user-asked questions worth keeping (type: synthesis)
index.md      catalog of every page, grouped by folder
log.md        append-only chronological log of operations
schema.md     this file
```

Create a folder on first use; don't pre-create empty ones.

> Naming note: `raw/` holds the original sources. `summaries/` holds LLM-written **summary pages** that point into `raw/` via `source_ref`. Don't confuse "source" (the raw original) with "summary" (the wiki page about it).

## Page conventions

Every wiki page has YAML frontmatter:

```yaml
---
type: summary | entity | concept | comparison | overview | synthesis
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [[summary-page-1]], [[summary-page-2]]   # for entity/concept/synthesis — links to the summary pages that cite it
source_ref: raw/path/to/file                       # for summary pages only — always a vault-relative path under raw/
external_origin: /absolute/path/outside/vault      # optional; summary pages only; preserves the original location when copied in from outside
---
```

Summary pages must have an actual copy of their source under `raw/`. A pointer to an external path or URL alone is not enough.

Body guidelines:
- Use `[[Wikilinks]]` for every mention of another wiki page. Obsidian resolves them.
- Top of body: one-paragraph summary suitable for the index.
- For `summary` pages: key takeaways + short quote block for load-bearing claims.
- For `entity`/`concept` pages: definition, attributes, relations, citations as `[[summary-page]]`.
- For `comparison` pages: the items being compared, a table or side-by-side, conclusions, citations.
- For `overview` pages: a bird's-eye map (domain diagram or layered structure), entry-points into other pages. Meant to be read first.
- For `synthesis` pages: the question being answered, the answer, evidence with citations.
- When a new source contradicts an existing claim, keep both and add a `## Contradictions` block noting each side + source.

## File naming

- Kebab-case, descriptive: `transformer-architecture.md`, `andrej-karpathy.md`.
- Summary pages mirror the source name: `raw/foo.pdf` → `summaries/foo.md`.

## index.md

Grouped by folder. Each line: `- [[page-name]] — one-line summary (N sources)`. Rebuilt on every ingest.

## log.md

Append-only. Entry format:
```
## [YYYY-MM-DD] <op> | <Title>
- touched: [[page1]], [[page2]]
- notes: short description
```
Ops: `ingest`, `query`, `lint`, `manual`.

## When in doubt

- Prefer updating existing pages over creating new ones.
- Prefer linking over duplicating.
- Flag uncertainty explicitly (`> [!note] Unverified:`) rather than silently omitting.
- Never edit anything under `raw/`.
