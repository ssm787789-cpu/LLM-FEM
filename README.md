# LLM-FEM — OptumGX Scripting Wiki

A persistent, compounding knowledge base for scripting **OptumGX** (a geotechnical FEM
program with a Python/gRPC API), built with the **LLM-Wiki pattern** by
[Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

## What this is

OptumGX exposes a large Python API — an object hierarchy (`GX → Project → Model → Stage`),
~27 material models, dozens of load/boundary-condition features, and a dense result model
— spread across ~90 library files plus two coexisting API generations (V1 and V2). Every
time you write a script you would otherwise re-derive the same facts from source code and
manuals.

This repository turns that one-time reading effort into a **durable, interlinked wiki**:
LLM-generated Markdown pages that explain *how to code each thing*, cross-reference each
other, and already have the contradictions and gotchas flagged. The wiki gets richer with
every source ingested and every question answered — it **compounds** instead of evaporating
into chat history.

**Problem it solves:** no more digging through manuals or decompiled API files every time
you script an analysis. Ask a question, get an answer grounded in the actual shipped code,
with working snippets labeled `VERBATIM` or `INFERRED`.

## Three-layer architecture

```
Raw Sources (READ-ONLY)        Wiki (LLM-owned)              Schema (config)
C:\Users\Public\OPTUM CE\  →   optumgx-wiki/wiki/        ←   optumgx-wiki/CLAUDE.md
the shipped OptumGX install     interlinked Markdown          rules & workflows that
(library/OptumGX, plugins)      pages, never deleted          make Claude a disciplined
immutable source of truth       only by the assistant         wiki maintainer
```

1. **Raw Sources** — the immutable OptumGX installation. Read, never modified.
2. **The Wiki** — LLM-generated, interlinked pages with YAML frontmatter and
   `[[wikilinks]]`. The assistant owns this layer.
3. **The Schema** (`CLAUDE.md`) — defines page format, tag taxonomy, ingest/query/lint
   workflows, and absolute rules. This is what keeps the wiki disciplined rather than ad-hoc.

## Directory structure

```
optumgx-wiki/
├── CLAUDE.md            # Schema: rules & workflows for maintaining the wiki
├── README.md           # This file
├── index.md            # Content catalog — read first before any query
├── log.md              # Append-only chronological record (grep-parseable)
├── overview.md         # High-level synthesis of the whole API
├── raw/                # Pointer to the READ-ONLY raw sources
└── wiki/
    ├── api/            # One page per API class / function group (15 pages)
    ├── concepts/       # FEM & API concepts as used in OptumGX (10 pages)
    ├── workflows/      # Common scripting patterns, step by step (7 pages)
    └── examples/       # Annotated, working code snippets (3 pages)
```

Every page carries frontmatter (`title`, `category`, `source`, `tags`, `source_count`,
`relates_to`) and links to siblings with `[[wikilinks]]` — so it is also a navigable
**Obsidian** vault (open `optumgx-wiki/` as a vault; graph hubs = most important classes).

## How to use it with Claude Code

The wiki is designed to be *used and extended* by Claude Code in this directory.

- **Ask a scripting question.** Claude reads `index.md` first, opens the relevant pages,
  and answers with `[[citations]]` and labeled code. Good answers get filed back as new
  pages, so the wiki improves as you use it.
  > "How do I run a factor-of-safety analysis and read the FOS?"
  > "How do I give Young's modulus a depth gradient?"

- **Ingest a new source.** Point Claude at a new file under the raw sources and ask it to
  ingest it. It follows the ingest workflow in `CLAUDE.md`: read fully → write/update pages
  → update `index.md` → append to `log.md` → commit `ingest: <filename>`.

- **Lint periodically.** Ask Claude to run the lint checklist in `CLAUDE.md` to find
  contradictions, orphan pages, stale claims, and missing cross-references.

- **Browse manually.** Open the repo as an Obsidian vault, or just start at
  [`overview.md`](overview.md) and [`index.md`](index.md).

See [`CLAUDE.md`](CLAUDE.md) for the full rules, page format, and workflows.

## Credit

Based on the LLM-Wiki pattern by Andrej Karpathy:
<https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>
