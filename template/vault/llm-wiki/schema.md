---
type: schema
updated: 2026-04-15
---

# Wiki Schema

Conventions the LLM follows when maintaining this wiki. Co-evolve this file with the user over time.

## Three layers

1. **Raw sources** (`raw/`) — immutable. LLM reads but never modifies. Articles, papers, PDFs, clippings. Images go in `raw/assets/`.
2. **Wiki** (everything else) — LLM-owned. Summaries, entity pages, concept pages, syntheses. Cross-linked with Obsidian wikilinks.
3. **Schema** (this file + `../../.claude/`) — how the wiki is structured and how the LLM behaves.

## Directory layout

```
raw/          sources as originally captured (immutable)
  assets/     downloaded images
sources/      one summary page per raw source
entities/     people, orgs, places, products — anything that can be "about"
concepts/     ideas, topics, techniques
syntheses/    comparisons, analyses, user-asked questions worth keeping
index.md      catalog of every page, grouped by folder
log.md        append-only chronological log of operations
schema.md     this file
```

Create a folder on first use; don't pre-create empty ones.

## Page conventions

Every wiki page has YAML frontmatter:

```yaml
---
type: source | entity | concept | synthesis
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [[source-page-1]], [[source-page-2]]   # for entity/concept/synthesis
source_ref: raw/path/to/file                    # for source pages only
---
```

Body guidelines:
- Use `[[Wikilinks]]` for every mention of another wiki page. Obsidian resolves them.
- Top of body: one-paragraph summary suitable for the index.
- For `source/` pages: key takeaways + short quote block for load-bearing claims.
- For `entity`/`concept` pages: definition, attributes, relations, source citations as `[[source-page]]`.
- For `synthesis` pages: the question being answered, the answer, evidence with citations.
- When a new source contradicts an existing claim, keep both and add a `## Contradictions` block noting each side + source.

## File naming

- Kebab-case, descriptive: `transformer-architecture.md`, `andrej-karpathy.md`.
- Source pages mirror the source name: `raw/foo.pdf` → `sources/foo.md`.

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
