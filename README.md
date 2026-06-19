# LLM-FEM — An LLM-Wiki for OptumGX Scripting

A persistent, interlinked knowledge base for scripting **OptumGX**, built with the
**LLM-Wiki pattern** by
[Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

## What is OptumGX?

[OptumGX](https://optumce.com/) is a geotechnical **Finite Element Method (FEM)** software
by Optum CE, used for geotechnical analyses such as **slope stability, bearing capacity,
consolidation, seepage, and excavation**. Beyond its GUI, OptumGX ships a **Python / gRPC
API** that lets you build models, assign materials, apply loads and boundary conditions,
run analyses, and extract results entirely from scripts — ideal for automation and
parametric studies.

## What is this repository?

The OptumGX Python API is large: an object hierarchy (`GX → Project → Model → Stage`),
~27 material models, dozens of load and boundary-condition features, and a dense result
model — spread across many source files, with two coexisting API generations (V1 and V2).
Writing a script usually means re-deriving the same facts from source code and manuals.

**LLM-FEM** turns that one-time reading effort into a **durable, compounding wiki**:
LLM-generated Markdown pages that explain *how to code each thing*, cross-reference each
other, and already have the contradictions and gotchas flagged. Instead of reverse-
engineering the API again, you **query the wiki** and get answers grounded in the actual
shipped code, with working snippets labeled `VERBATIM` (copied from source) or `INFERRED`
(reconstructed from the API surface).

The wiki gets richer with every source ingested and every question answered — it
**compounds** instead of evaporating into chat history.

## Who is this for?

Geotechnical engineers and researchers who use OptumGX and want to **automate analyses via
Python scripting** — running design sweeps, batch analyses, or custom result extraction —
without repeatedly reverse-engineering the API.

## Three-layer architecture

```
Raw Sources (READ-ONLY)                Wiki (LLM-owned)            Schema (config)
<your OptumGX installation dir>  →     optumgx-wiki/wiki/      ←   optumgx-wiki/CLAUDE.md
the shipped OptumGX install            interlinked Markdown        rules & workflows that
(its Python library + plugins)         pages, never deleted        make the agent a
immutable source of truth              only by the assistant       disciplined maintainer
```

1. **Raw Sources** — your immutable OptumGX installation. Read, never modified.
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
`relates_to`) and links to siblings with `[[wikilinks]]` — so the repository also works as
a navigable **Obsidian** vault (graph hubs = the most important API classes).

## Prerequisites

- **OptumGX installed**, with the Python API available (the GUI acts as the local API
  server).
- **[Claude Code](https://claude.ai/code)** — or any LLM agent that can read and write
  files — to query, ingest, and maintain the wiki.
- **[Obsidian](https://obsidian.md/)** *(optional)* — to browse the wiki as an interlinked
  graph.

## Getting started

1. **Clone this repository**
   ```bash
   git clone https://github.com/ssm787789-cpu/LLM-FEM.git
   cd LLM-FEM
   ```
2. **Point the schema at your OptumGX install.** Edit `CLAUDE.md` and replace the raw
   source path (the source-of-truth map) with *your* local OptumGX installation directory.
3. **Open Claude Code in this directory** (or your preferred LLM agent).
4. **Ingest the API sources.** Ask the agent to ingest the OptumGX Python library,
   following the ingest workflow in `CLAUDE.md` (read fully → write/update pages → update
   `index.md` → append to `log.md` → commit).
5. **Start querying**, e.g.:
   > "How do I run a factor-of-safety analysis and read the FOS?"
   > "How do I create a Mohr-Coulomb soil and assign it to a face?"
   > "How do I give Young's modulus a depth gradient?"

## How to use it with Claude Code

**Claude Code** is an agentic coding assistant by Anthropic that can read, write, and
maintain files autonomously. The wiki is designed to be used and extended by it (or a
similar agent) in this directory.

- **Ask a scripting question.** The agent reads `index.md` first, opens the relevant pages,
  and answers with `[[citations]]` and labeled code. Good answers get filed back as new
  pages, so the wiki improves as you use it.
- **Ingest a new source.** Point the agent at a new file under your raw sources and ask it
  to ingest it, following the workflow in `CLAUDE.md` and committing `ingest: <filename>`.
- **Lint periodically.** Ask the agent to run the lint checklist in `CLAUDE.md` to find
  contradictions, orphan pages, stale claims, and missing cross-references.
- **Browse manually.** Open the repo as an Obsidian vault, or start at
  [`overview.md`](overview.md) and [`index.md`](index.md).

See [`CLAUDE.md`](CLAUDE.md) for the full rules, page format, and workflows.

## Credit

Based on the LLM-Wiki pattern by Andrej Karpathy:
<https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>
