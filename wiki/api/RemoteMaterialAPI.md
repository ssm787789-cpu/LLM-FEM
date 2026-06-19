---
title: RemoteMaterialAPI
category: api
source: Materials/RemoteMaterialAPI.py
last_updated: 2026-06-19
tags: [#material, #script, #v2, #automation]
source_count: 1
relates_to:
  - page: "[[Project]]"
    rel: required_by
  - page: "[[RemoteMaterial]]"
    rel: supports
  - page: "[[MohrCoulomb (material)]]"
    rel: example_of
  - page: "[[Material Models]]"
    rel: example_of
  - page: "[[Assign Materials]]"
    rel: example_of
---

# RemoteMaterialAPI

## Summary
`RemoteMaterialAPI` is the **material factory** mixed into [[Project]] (and [[GxClient]]).
It gives one constructor method per material model plus generic getters. Calling a
constructor **creates the material on the server immediately** and returns a live
[[RemoteMaterial]] whose properties auto-sync on assignment.

## Details
Defined in `Materials/RemoteMaterialAPI.py`.

### Construction
`prj.<MaterialName>(name=..., **params)` — e.g. `prj.MohrCoulomb(...)`,
`prj.LinearElastic(...)`, `prj.GeneralPlate(...)`. All parameters are keyword,
default `...` (Ellipsis = "leave at server default"). Internally each constructor caches
the kwargs then calls `obj._create()` (see [[RemoteMaterial]]).

**Available constructors** (the `MaterialType` union):
`MohrCoulomb`, `MohrCoulombEngineering`, `DruckerPrager`, `AUS`, `HardeningSoil`, `HMC`,
`Rigid`, `Tresca`, `NailRow`, `NGIADP`, `ModifiedCamClay`, `MultiMohrCoulomb`,
`LinearElastic`, `HoekBrown`, `ReinforcedConcrete`, `FlatPlateConcrete`, `GeneralPlate`,
`SheetPile`, `PileRow`, `Beam`, `RigidBeam`, `Water`, `Geogrid`, `RigidPlate`,
`FlatPlateSteel`, `Connector`. See [[Material Models]] for what each is for.

### Retrieval
| Method | Returns |
|--------|---------|
| `get_material(name)` | The single typed material (dispatched via the proto `oneof`). |
| `get_material()` | List of all materials. |
| `get_<Type>(name)` | Typed getter that also type-checks (e.g. `get_MohrCoulomb`). |
| `get_<category>_material(name=None)` | By `MaterialCategory`: solid, fluid, shell, plate, shearjoint, beam, connector, geogrid, pilerow, nailrow. |

Material **type** is recovered from `IMaterialData.WhichOneof('material_data')` and mapped
back to the Python class via `_MATERIAL_ONEOF_MAP`.

## Code Example
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py)
Soil = prj.MohrCoulomb(name='Soil', c=c, phi=phi, gamma_dry=gamma, color=rgb(180,180,90))
Wall = prj.GeneralPlate(name='Wall', m_px=mp, color='blue')
```
```python
# INFERRED — fetch and tweak an existing material
clay = prj.get_MohrCoulomb('Soil')   # type-checked getter
clay.phi = 28.0                       # auto-syncs to server
```

## Related
- [[Project]] — exposes these constructors (`prj.MohrCoulomb(...)`).
- [[RemoteMaterial]] — the base class of every returned object (sync/batch/delete).
- [[MohrCoulomb (material)]] — a worked per-material parameter reference.
- [[Material Models]] — catalog and selection guidance.
- [[Assign Materials]] — workflow tying materials to shapes.

## Contradictions
Material module docstrings (e.g. `Materials/MohrCoulomb.py`) show
`gx.new.MohrCoulomb(name=..., phi=30.0)`, but **no `new` attribute exists** in the
shipped `library/OptumGX/` package — `grep` finds no `new` accessor on `GX`, `Project`,
`GxClient`, or `RemoteMaterialAPI`. The working call in shipped plugin code is
`prj.MohrCoulomb(...)` (factory method directly on the project). Treat `gx.new.<X>` as
**documentation drift / aspirational**; use `prj.<X>(...)`. Flagged for human review —
see [[Material Models]] Contradictions and `log.md`.

## Notes
- `color=` accepts an `rgb(r,g,b)` value or a named-color string (`'blue'`); see
  [[Colors and rgb]].
- Constructors create immediately; there is no separate "add to project" step.
