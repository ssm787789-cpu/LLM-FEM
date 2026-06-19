---
title: Extrude 2D to 3D
category: workflow
source: v2.py, DataModelV2.py
last_updated: 2026-06-19
tags: [#geometry, #mesh, #script, #v2]
source_count: 2
relates_to:
  - page: "[[Model]]"
    rel: example_of
  - page: "[[ShapeList]]"
    rel: supports
  - page: "[[Build 2D Geometry]]"
    rel: required_by
---

# Extrude 2D to 3D

## Summary
Build a 2D cross-section, set per-shape extrusion depths, then generate a 3D model from
it. OptumGX supports straight extrusion, revolution, and slicing 3D back to 2D.

## Details
- Per-shape extrusion settings via [[ShapeList]]:
  `sel.set_2d_to_3d_settings(depth_in, depth_out, repetition, repetition_spacing,
  repetition_out, repetition_in, fill, fill_in, fill_out, as_face)` or the live
  `sel.settings_3d.depth_in = ...` accessor.
- Generate the 3D model:
  - `mod.extrude_2d_to_3d_model(name)` — uniform/straight extrusion of the whole section.
  - `mod.revolve_2d_to_3d_model(angle_deg=180, N=12, name)` — axisymmetric-style sweep.
  - `mod.extrude(shapes, vector)` / `mod.revolve(...)` / `mod.extrude_along(shapes, path)`
    — extrude specific shapes.
- Reverse: `mod.slice_3d_to_2d_model(p0, p1, name)`.

## Steps
```python
# INFERRED — section -> per-edge depths -> 3D
mod.add_rectangle([0,0],[10,5])              # 2D section
sel = mod.select([5,2.5], types='face')
sel.set_2d_to_3d_settings(depth_in=10, depth_out=10, as_face=True)
m3d = mod.extrude_2d_to_3d_model('Model 3D') # new 3D Model
```
```python
# INFERRED — live settings accessor (VERBATIM-style from DataModelV2 docstring)
sel = mod.select([2,4], types='edge')
sel.settings_3d.depth_in = 10
print(sel.settings_3d.depth_out)
```

## Related
- [[Model]] — extrude/revolve/slice methods.
- [[ShapeList]] — `set_2d_to_3d_settings` / `settings_3d`.
- [[Build 2D Geometry]] — the section you extrude.

## Notes
- `extrude_2d_to_3d_model` returns a new [[Model]] (3D); the original 2D model is kept.
- `repetition`/`fill` controls build repeated bays (e.g. piled walls) along the extrusion.
- 3D models use `volume` shapes for solids and `face` shapes for plates — adjust your
  selection `types` accordingly after extrusion.
