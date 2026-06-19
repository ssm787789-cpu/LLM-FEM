---
title: V1 vs V2 API
category: concept
source: v2.py, GxClient.py, DataModel.py, Materials/RemoteMaterialAPI.py
last_updated: 2026-06-19
tags: [#v1, #v2, #script, #gotcha, #grpc]
source_count: 4
relates_to:
  - page: "[[GxClient]]"
    rel: supports
  - page: "[[GX]]"
    rel: supports
  - page: "[[Object Hierarchy]]"
    rel: supports
  - page: "[[Output]]"
    rel: contradicts
---

# V1 vs V2 API

## Summary
Two generations of the scripting API ship in the same package and share one gRPC channel.
**V2** (object hierarchy [[GX]]→[[Project]]→[[Model]]→[[Stage]]) is the current, preferred
surface. **V1** (flat [[GxClient]]) remains for back-compat and older example scripts.
Mixing their idioms is the most common source of confusion.

## Details

| Aspect | V1 | V2 |
|--------|----|----|
| Entry point | `GxClient(port=6000)` | `GX()` |
| Stub | `client._api` (`Contract_pb2`) | `client._v2` (`ContractV2_pb2`) |
| Model object | flat methods on the client | [[Project]]/[[Model]]/[[Stage]] |
| Materials | [[RemoteMaterialV1]] (explicit create/save) | [[RemoteMaterialAPI]] (auto-sync objects) |
| Analysis settings | `set_analysis_properties(...)` | attributes via [[AnalysisProperties]] + bulk call |
| Results | `gx.output()` **method** → `Output.Output` | `stage.output` **property** → `StageOutput` |
| Enums | `DataModel.py` (int enums) | `DataModelV2.py` / string enums |

Both stubs ride the same channel created by [[GxClient]]; `GX()` simply reuses
`client._v2`. So V2 is "V1's transport + a nicer object model."

## Code Example
```python
# INFERRED — same task, two generations
# V2 (preferred)
gx = GX(); prj = gx.create_project('p'); mod = prj.create_model('m')
mod.add_polygon([[0,0],[10,0],[10,5],[0,5]])

# V1 (legacy)
client = GxClient()
client.create_new_model('m')
client.add_polygon([[0,0],[10,0],[10,5],[0,5]])
```

## Related
- [[GX]] / [[GxClient]] — the two entry points.
- [[Object Hierarchy]] — the V2 structure.
- [[RemoteMaterialAPI]] / [[RemoteMaterialV1]] — material flavors.
- [[Output]] — documents the property-vs-method results contradiction.

## Contradictions
- `output`: V2 property vs V1 method — see [[Output]] Contradictions.
- `GX()` ignores a custom port (always builds `GxClient()` on 6000) — see [[GX]] Notes.
- `gx.new.<Material>` appears in material docstrings but is undefined in the shipped
  package — see [[RemoteMaterialAPI]] Contradictions.

## Notes
- Rule of thumb: **start every new script with `gx = GX()`** and stay in V2. Reach for V1
  only when adapting old code or when you need a non-default port (via raw `client._v2`).
