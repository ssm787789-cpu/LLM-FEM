---
title: RemoteMaterialV1
category: api
source: DataModel.py
last_updated: 2026-06-19
tags: [#material, #v1, #script, #gotcha]
source_count: 1
relates_to:
  - page: "[[GxClient]]"
    rel: required_by
  - page: "[[RemoteMaterialAPI]]"
    rel: superseded
  - page: "[[V1 vs V2 API]]"
    rel: example_of
---

# RemoteMaterialV1

## Summary
`RemoteMaterialV1` is the **legacy (V1) material creation** mixin on [[GxClient]]. Unlike
the V2 [[RemoteMaterialAPI]] (live auto-syncing objects), V1 materials are plain Python
objects you fill in and then `create_material(...)` / `save_material(...)` explicitly.
Use it only with V1 scripts; prefer [[RemoteMaterialAPI]] for new work.

## Details
Defined in `DataModel.py`. `create_material(material)` builds an `IMaterialRequest` from
the object's attributes (`bindAttr` copies the inheritance chain of proto messages) and
sends it over the V1 stub (`self._v1.create_material`). Supported V1 classes include
`MohrCoulomb`, `MohrCoulombEngineering`, `RigidSolid`, `Tresca`, `RigidShell`,
`VonMisesPlate`, `BarMaterial`, `NailRowMaterial`, `NGIADP`, `ModifiedCamClay`,
`DruckerPrager`, `AUS`, `HardeningSoil`, `HMC`, `MultiMohrCoulomb`, `LinearElastic`
(→ `IElasticSolid`).

After creation the object gets a `uuid`, a back-reference `_gx`, and a `sync()` method
(`_updateMaterial`) that re-saves it.

## Code Example
```python
# INFERRED — V1 explicit create/save flow
m = MohrCoulomb()       # plain object
m.phi = 30; m.c = 5
gx.create_material(m)   # now has m.uuid; lives on server
m.phi = 32
m.sync()                # push the edit (V1 has no auto-sync)
```

## Related
- [[GxClient]] — hosts these V1 methods.
- [[RemoteMaterialAPI]] — the V2 replacement (auto-syncing live objects).
- [[V1 vs V2 API]] — choosing between them.

## Notes
- **Gotcha:** V1 material names differ from V2 — V1 `RigidSolid`/`RigidShell`/
  `VonMisesPlate`/`BarMaterial`/`ElasticSolid` vs V2 `Rigid`/`RigidPlate`/`FlatPlateSteel`/
  `Connector`/`LinearElastic`. Don't mix vocabularies.
- No `batch()`; edits require an explicit `sync()`/`save_material()`.
