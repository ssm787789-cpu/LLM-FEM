---
title: Csys
category: api
source: v2.py
last_updated: 2026-06-19
tags: [#geometry, #load, #boundary, #script, #v2]
source_count: 1
relates_to:
  - page: "[[Project]]"
    rel: required_by
  - page: "[[StageBase]]"
    rel: supports
  - page: "[[FeatureAPI]]"
    rel: supports
---

# Csys

## Summary
`Csys` is a user-defined **coordinate system**. It is created on the [[Project]] and used
to express loads, supports, and boundary conditions in a non-global frame (e.g. a sloped
load, a rotated wall support). Selections and features reference it via a `csys=` argument.

## Details
Defined in `v2.py` (`class Csys`).

### Creation (on [[Project]])
- `create_csys_2d(origo=None, direction_i=None, name=None)`
- `create_csys_3d(origo=None, direction_i=None, direction_j=None, name=None)`
- `get_csys(name=None)` → a `Csys` or list.

### Editable properties (auto-update on assignment)
`name`, `origo`, `direction_i`, `direction_j`, `direction_k`.

### Activation
`StageBase.set_active_csys(csys | 'local' | 'global 2d' | 'global 3d')` and
`get_active_csys()`. Built-in frame names accepted as strings: `local`, `global 2d`,
`global 3d` (and underscore/space variants).

## Code Example
```python
# INFERRED — a 2D csys rotated to a slope, used for a tangential load
slope = prj.create_csys_2d(origo=[0,0], direction_i=[1, 0.5], name='slope')
edge  = mod.select([3, 1.5], types='edge')
mod.set_line_load(edge, p=-15, direction='x', csys=slope)
```

## Related
- [[Project]] — creates and stores coordinate systems.
- [[StageBase]] — activate a csys for subsequent operations.
- [[FeatureAPI]] — loads/BCs take a `csys=` argument.

## Notes
- `getCsysId` (in `DataModelV2.py`) resolves a `Csys` object, a built-in frame string, or
  `None` to the id the server expects; invalid strings raise `ValueError`.
- Coordinate-type on loads (`CoordinateType` = `local | Global | UserDefined`) is distinct
  from the `Csys` object — `UserDefined` pairs with an explicit `csys=`.
