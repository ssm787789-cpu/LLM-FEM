---
title: Build 2D Geometry
category: workflow
source: v2.py, plugins/Example_Plugin/example_plugin.py
last_updated: 2026-06-19
tags: [#geometry, #selection, #script, #v2]
source_count: 2
relates_to:
  - page: "[[Model]]"
    rel: example_of
  - page: "[[Shapes and Selection]]"
    rel: supports
  - page: "[[Assign Materials]]"
    rel: required_by
---

# Build 2D Geometry

## Summary
Construct a 2D domain on a [[Model]] by drawing boundary lines or primitives, then select
the resulting faces/edges to act on. OptumGX auto-creates the interior face once a region
is closed.

## Details
Two common styles:
1. **Boundary lines** — draw the outline with `add_line` (flexible, irregular domains).
2. **Primitives** — `add_rectangle`, `add_circle`, `add_polygon`, `add_arc` (fast for
   regular shapes).

After construction, **select** with [[Shapes and Selection|selection]] to get the
[[ShapeList]] handles you pass to materials and features.

## Steps
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py) — closed boundary by lines
H = 5; D = 3
mod.add_line([0,0],[0,H])
mod.add_line([0,H],[2*H,H])
mod.add_line([2*H,H],[2*H,-2*H])
mod.add_line([2*H,-2*H],[-2*H,-2*H])
mod.add_line([-2*H,-2*H],[-2*H,0])
mod.add_line([-2*H,0],[0,0])
mod.add_line([0,0],[0,-D])          # internal wall line
sel = mod.select([0,0], types='face')   # interior face auto-created
mod.zoom_all()
```
```python
# INFERRED — same domain via a primitive
mod.add_rectangle([-10,-10],[10,5])
soil = mod.select([0,0], types='face')
```

## Related
- [[Model]] — all geometry methods.
- [[Shapes and Selection]] / [[ShapeList]] — selecting what you built.
- [[Assign Materials]] — the next step.
- [[Extrude 2D to 3D]] — turning this into a 3D model.

## Notes
- Coordinates are `[x, y]` in 2D (`[x, y, z]` in 3D). Units are whatever the model uses.
- Re-select after edits; shapes are positional, not named ([[Shapes and Selection]]).
- `add_polygons([...])` is faster than many `add_polygon` calls for bulk import.
