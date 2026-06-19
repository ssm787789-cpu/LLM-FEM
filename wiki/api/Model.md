---
title: Model
category: api
source: v2.py
last_updated: 2026-06-19
tags: [#geometry, #script, #v2, #stage, #mesh]
source_count: 1
relates_to:
  - page: "[[Project]]"
    rel: required_by
  - page: "[[Stage]]"
    rel: required_by
  - page: "[[StageBase]]"
    rel: extends
  - page: "[[FeatureAPI]]"
    rel: extends
  - page: "[[Build 2D Geometry]]"
    rel: example_of
---

# Model

## Summary
`Model` is where **geometry is built** and the base ("model") analysis settings live.
A model owns a sequence of [[Stage]] objects for staged construction. It is the V2
object you spend most scripting time in: adding lines/polygons/solids, selecting
shapes, assigning materials and [[FeatureAPI|features]], then creating stages.

## Details
Defined in `v2.py` as `class Model(StageBase, ModelFeatureAPI)`. It therefore has:
- all geometry/selection/output from [[StageBase]],
- all loads/BCs/structural features from [[FeatureAPI]] (`ModelFeatureAPI` variant),
- geometry-construction methods unique to `Model` (below).

### Geometry construction (2D and 3D)
| Method | Notes |
|--------|-------|
| `add_line(p0, p1)`, `add_lines(points)`, `add_polyline(points)` | Lines. |
| `add_polygon(points)`, `add_polygons(points)` | Faces (3D-capable). `add_polygons` is faster than looping. |
| `add_vertex(p0)`, `add_arc(p0,p1,p2,N)`, `add_circle(center,radius,N)` | |
| `add_rectangle(p0,p1)`, `add_box(p0,p1)` | Quad / box. |
| `add_sphere(center,radius,N)`, `add_nprism(...)`, `add_ncone(...)`, `add_prism(points,height)` | 3D primitives. |
| `initialize_geometry(points, triangles)` | Bulk mesh-style import. |

### Edit / transform
`move`, `copy`, `mirror`, `scale`, `rotate`, `rectangular_array`, `polar_array`,
`set_vertex_position`, `delete_shapes`, `delete_interior`, `hide_shapes`/`unhide_shapes`,
transparency toggles, `set_hinge_2d`/`set_hinge_3d`.

### 2D→3D and 3D→2D
`extrude(shapes, vector)`, `extrude_along(shapes, path)`, `revolve(...)`,
`extrude_2d_to_3d_model(name)`, `revolve_2d_to_3d_model(angle_deg, N, name)`,
`slice_3d_to_2d_model(p0, p1, name)`. See [[Extrude 2D to 3D]].

### Stages
`get_stage(name=None)`, `get_current_stage()`, `set_current_stage(name_or_stage)`,
and `create_stage(name)` (from [[StageBase]]). See [[Staged Construction]].

### Properties
`model_type` → `ModelType`; `from_model` (the model this one was derived from).

## Code Example
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py) — build a 2D domain by lines
mod = prj.create_model('FOS')
H = 5; D = 3
mod.add_line([0,0],[0,H])
mod.add_line([0,H],[2*H,H])
mod.add_line([2*H,H],[2*H,-2*H])
mod.add_line([2*H,-2*H],[-2*H,-2*H])
mod.add_line([-2*H,-2*H],[-2*H,0])
mod.add_line([-2*H,0],[0,0])
sel = mod.select([0,0], types='face')   # pick the enclosed face
mod.set_solid(sel, Soil)
mod.set_standard_fixities()
mod.zoom_all()
```

## Related
- [[Project]] — creates models and owns the materials you assign.
- [[StageBase]] — geometry-agnostic selection/output/analysis methods inherited here.
- [[Stage]] — child analysis phases created from a model.
- [[FeatureAPI]] — `set_point_load`, `set_plate`, `set_water_table`, etc.
- [[Build 2D Geometry]], [[Extrude 2D to 3D]] — workflows.

## Notes
- Geometry-construction methods live on `Model`, **not** on [[Stage]]; stages inherit the
  model's geometry and only change analysis settings / staged feature activation.
- `set_standard_fixities()` and `set_solid()` come from `ModelFeatureAPI`/[[StageBase]].
- Closing a face by drawing its bounding lines (as above) auto-creates the interior
  face, which `select(..., types='face')` then picks.
