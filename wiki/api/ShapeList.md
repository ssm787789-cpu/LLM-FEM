---
title: ShapeList
category: api
source: DataModelV2.py
last_updated: 2026-06-19
tags: [#selection, #geometry, #script, #v2]
source_count: 1
relates_to:
  - page: "[[StageBase]]"
    rel: required_by
  - page: "[[Shapes and Selection]]"
    rel: example_of
  - page: "[[FeatureAPI]]"
    rel: supports
---

# ShapeList

## Summary
`ShapeList` (and its element `Shape`) is the **handle you pass everywhere** in the V2
API — to assign materials, apply loads/BCs, transform geometry, and read per-element
results. Every selection method on [[StageBase]] returns a `ShapeList`. It is a thin,
stage-bound wrapper over geometric entities (vertices/edges/faces/volumes).

## Details
Defined in `DataModelV2.py`.

### `Shape`
Fields: `id` (int), `type` ([[Shapes and Selection|ShapeType]]), and lazily populated
`length`, `area`, `volume`, `location`. Sub-entity helpers: `vertices()`, `edges()`,
`faces()`, `volumes()`. Each `Shape` is bound to its stage/model (`_stage`).

### `ShapeList`
- `.values` — the underlying list of `Shape`.
- Single-element convenience proxies (return the lone shape's value, else `None`):
  `.id`, `.type`, `.length`, `.area`, `.volume`, `.location`, and `.vertices()/.edges()/.faces()/.volumes()`.
- Set ops: `find(shape)`, `contains(shape)`, `remove(shape|list)`, `merge(other)`
  (must share the same stage), `get(i)` / `sl[i]` → single-element `ShapeList`, `len()`.
- `settings_3d` / `set_2d_to_3d_settings(...)` / `get_2d_to_3d_settings()` — per-shape
  extrusion controls used by [[Extrude 2D to 3D]].

`ShapeType` (in `DataModelV2.py`): `vertex=0, edge=1, face=2, volume=3, void=4`. It is an
`IntStrEnum`, so `'face'`, `2`, and `ShapeType.face` all compare equal — APIs accept the
string form (`types='face'`).

## Code Example
```python
# INFERRED — pick, inspect, and combine selections
faces = mod.faces()                     # ShapeList of all faces
soil  = mod.select([1,1], types='face') # single face -> ShapeList(len 1)
print(soil.area, soil.id)               # convenience props (only valid when len==1)

left  = mod.select([0,2], types='edge')
right = mod.select([10,2], types='edge')
both  = left.merge(right)               # same stage required
mod.set_support(both)                   # pass the list straight to a feature
```

## Related
- [[StageBase]] — every `select`/`get_shapes` call returns a `ShapeList`.
- [[Shapes and Selection]] — the selection concept and `ShapeType`.
- [[FeatureAPI]] — consumes `ShapeList` for loads/BCs/structural elements.

## Notes
- The single-element convenience properties return `None` when the list has ≠1 element —
  guard with `len(sel)==1` before reading `.area`/`.id` in loops.
- `merge` raises if the two lists come from different stages — selections are
  stage-bound, not global.
- There is also a separate, simpler `ShapeList` in the V1 `DataModel.py`; the V2 one
  (stage-aware, with `_stage`) is the current surface. See [[V1 vs V2 API]].
