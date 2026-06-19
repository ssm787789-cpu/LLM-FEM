---
title: StageBase
category: api
source: v2.py
last_updated: 2026-06-19
tags: [#selection, #geometry, #solver, #output, #script, #v2]
source_count: 1
relates_to:
  - page: "[[Model]]"
    rel: required_by
  - page: "[[Stage]]"
    rel: required_by
  - page: "[[ShapeList]]"
    rel: supports
  - page: "[[AnalysisProperties]]"
    rel: extends
  - page: "[[Shapes and Selection]]"
    rel: example_of
---

# StageBase

## Summary
`StageBase` is the shared base class of both [[Model]] and [[Stage]] in the V2 API. It
provides everything that is common to "something you can select shapes on and read
results from": **selection, shape queries, solver settings, run flags, coordinate
systems, and output**. If a method works on both a model and a stage, it is defined here.

## Details
Defined in `v2.py` as `class StageBase(v2gen._RemoteStage, AnalysisProperties)`.

### Selection & shapes → [[ShapeList]]
| Method | Purpose |
|--------|---------|
| `select(p0, p1=None, types=None, option='blue')` | Select by point or bounding box. `types` ∈ `'vertex'/'edge'/'face'/'volume'` (or `ShapeType`). `option` `'blue'`=exclusive, `'green'`=inclusive. |
| `get_shapes(types=None)` | All shapes (optionally filtered). |
| `vertices()/edges()/faces()/volumes()` | Convenience filters. |
| `get_sub_shapes(shapes, sub_types)` | Child shapes (e.g. edges of a face). |
| `get_shapes_by_vertices(type, vertices)` | Shapes incident to vertices. |
| `get_shape_by_id(id)` | Lookup by integer id. |
| `get_shared_edges(a,b)` / `get_shared_faces(a,b)` | Topological intersection. |
| `get_vertices(points)` | Vertices at given coordinates. |
| `get_selected_shapes()` | Current GUI selection. |

### Solver & run control
- `set_solver_settings(solver_type, solver_method=None, equilibrate=None)` —
  `solver_method` ∈ `Mosek | Clarabel | QDLDL | FAER | MKL | Panua`.
- `set_run_flag(RunFlag.run | RunFlag.none)` / `get_run_flag()`.

### Coordinate systems
`set_active_csys(csys)`, `get_active_csys()` → [[Csys]].

### Output
`output` (property) → see [[Output]].

### Misc
`zoom_all()`, `undo()`, `redo()`, `clone(name)`, `delete()`, `create_stage(name)`.

### Analysis settings
Inherits [[AnalysisProperties]] — see that page for the assignable settings.

## Code Example
```python
# INFERRED — selection patterns
faces = mod.faces()                        # all faces
f     = mod.select([2, 2], types='face')   # face containing point (exclusive)
allv  = mod.select([0,0], [10,10], types='vertex', option='green')  # box, inclusive
top   = mod.select([5, 5], types='edge')   # edge at a point
print(f.id, f.area)                         # ShapeList convenience props
```

## Related
- [[Model]] / [[Stage]] — the two concrete subclasses.
- [[ShapeList]] — return type of every selection method.
- [[AnalysisProperties]] — settings mixed in here.
- [[Output]] — `.output` results.
- [[Shapes and Selection]] — concept page on the selection model.

## Notes
- `select` overloads on the second argument: if it is a `ShapeType`/str it is treated as
  `types` (point selection); otherwise it is the opposite corner of a box.
- "blue" vs "green" mirrors the GUI's east/west box-selection colors:
  blue = `inclusive=True` flag (exclusive crossing), green = inclusive.
