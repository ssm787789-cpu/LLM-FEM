# CLAUDE.md — OptumGX Scripting Wiki Schema

This file is the **schema** that turns Claude into a disciplined maintainer of the
OptumGX scripting wiki, following the LLM-Wiki pattern (Raw Sources → Wiki → Schema).
Read this before doing anything else in this repo.

## Mission

Maintain a persistent, compounding **scripting reference** for the OptumGX Python API
(a gRPC client for the OptumGX geotechnical FEM program). Primary goal: answer
*"how do I code this?"* — prefer scripting utility over FEM theory.

## Three layers

1. **Raw Sources (READ-ONLY)** — `C:\Users\Public\OPTUM CE\OPTUM GX\`
   The shipped OptumGX install. The scripting-relevant sources are the Python package
   `library/OptumGX/` and the example `plugins/`. Never modify these.
2. **The Wiki (we own this)** — `wiki/` — interlinked Markdown pages.
3. **The Schema** — this file.

## Source-of-truth map (where the API actually lives)

| Area | File(s) under `library/OptumGX/` |
|------|----------------------------------|
| V2 object model (GX/Project/Model/Stage) | `v2.py` |
| V2 transport + V1 client | `GxClient.py` |
| V1 legacy object model + `RemoteMaterialV1` | `DataModel.py` |
| Enums, `Shape`, `ShapeList`, `rgb`, field wrappers | `DataModelV2.py`, `Common.py`, `ContractV2_enum.py` |
| Remote-sync base (cache/dirty/batch) | `RemoteObject.py` |
| Stage analysis settings (descriptor properties) | `RemoteStage.py` (class `AnalysisProperties`) |
| Material factory + per-type constructors | `Materials/RemoteMaterialAPI.py` |
| Material base class | `Materials/RemoteMaterial.py` |
| One file per material model | `Materials/<Name>.py` |
| Loads / BCs / structural features | `RemoteFeatures/FeatureAPI.py` + `RemoteFeatures/<Name>.py` |
| Result extraction (dense mesh) | `Output.py`, `Result.py`, `ResultV2.py`, `ResultData_pb2.py` |
| gRPC/proto transport (generated) | `Contract*_pb2*.py`, `ContractV2*_pb2*.py` |

The `*_pb2*.py` files are generated protobuf/gRPC stubs — document them only as the
**transport layer**, do not create a page per generated message.

## Wiki page format (every page in `wiki/`)

```yaml
---
title: <Page Title>
category: api | concept | workflow | example
source: <original filename(s)>
last_updated: <YYYY-MM-DD>
tags: [#tag1, #tag2]
source_count: <number of sources that informed this page>
relates_to:
  - page: "[[Target Page]]"
    rel: supports | extends | supersedes | contradicts | required_by | example_of
---
```

Body sections: `## Summary`, `## Details`, `## Code Example`, `## Related`,
`## Contradictions` (only if needed), `## Notes`.

## Tag taxonomy

`#material #boundary #stage #phase #solver #output #mesh #load #seepage`
`#consolidation #stability #excavation #pile #anchor #script #automation #gotcha`
Plus useful additions for this API: `#geometry #selection #grpc #v1 #v2 #result #plate`.

## Wikilinks

Use `[[Page Title]]` in prose **and** in the `relates_to` frontmatter. Every page must
have at least one inbound or outbound `[[wikilink]]`. A `[[link]]` to a not-yet-written
page is allowed — it marks a gap (see lint).

## Code-label rule (ABSOLUTE)

Every code block is labeled `# VERBATIM (<file>)` if copied exactly from a source, or
`# INFERRED` if reconstructed from the API surface. Never label inferred code verbatim.
Most "how do I" snippets are INFERRED; docstring examples and plugin code are VERBATIM.

## Ingest workflow

1. Read the source file fully.
2. Note key scripting takeaways.
3. Create/update pages in `wiki/`.
4. Update `index.md`.
5. Append to `log.md`: `## [YYYY-MM-DD] ingest | <filename>`.
6. `git commit -m "ingest: <filename>"`.

A single source may touch 10–15 pages — expected.

## Query workflow

1. Read `index.md` first.
2. Read the relevant pages.
3. Answer with `[[citations]]`.
4. If the answer is valuable, file it back as a new `wiki/` page and log it.

## Lint checklist (run periodically)

- [ ] Contradictions between pages (flag, never silently resolve).
- [ ] Stale claims superseded by newer sources.
- [ ] Orphan pages (no inbound `[[wikilinks]]`).
- [ ] Concepts mentioned but lacking their own page.
- [ ] Missing cross-references.
- [ ] Data gaps fillable from OptumGX online docs.

Append `## [YYYY-MM-DD] lint | <summary>` to `log.md`.

## Absolute rules

1. `C:\Users\Public\OPTUM CE` is READ-ONLY.
2. Every page has ≥1 `[[wikilink]]`.
3. Every snippet labeled VERBATIM or INFERRED.
4. Contradictions flagged and preserved.
5. Good answers filed back into the wiki.
6. `log.md` entries start with `## [YYYY-MM-DD]`.
7. `git commit` after every ingest.
8. Read `index.md` before answering a query.
9. Prefer scripting utility over theory.

## Domain quick-orientation (so pages stay consistent)

- The OptumGX **GUI app is the gRPC server**; the Python package is a **client** that
  must connect to a running instance on `localhost:6000` (default). Scripts mutate the
  live model you see in the GUI.
- Two API generations coexist: **V1** (`GxClient`, `Contract_pb2`) and **V2**
  (`GX`/`Project`/`Model`/`Stage`, `ContractV2_pb2`). **Prefer V2** for new scripts.
- Object hierarchy (V2): `GX → Project → Model → Stage`; materials & csys live on the
  `Project`; geometry on the `Model`; analysis settings on `Stage`/`Model`.
- Property writes on remote objects (materials, stage settings) **auto-sync** to the
  server on assignment; wrap multiple writes in `with obj.batch():` to sync once.
