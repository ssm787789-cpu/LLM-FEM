---
title: Shapes and Selection
category: concept
source: DataModelV2.py, v2.py
last_updated: 2026-06-19
tags: [#selection, #geometry, #script, #v2]
source_count: 2
relates_to:
  - page: "[[ShapeList]]"
    rel: supports
  - page: "[[StageBase]]"
    rel: supports
  - page: "[[Build 2D Geometry]]"
    rel: example_of
---

# Shapes and Selection

## Summary
Everything you act on in a model — assigning a material, applying a load, deleting
geometry — is addressed by **selecting shapes**. A shape is a vertex, edge, face, or
volume; selection returns a [[ShapeList]]. Getting selection right is most of the work in
a geometry script, because OptumGX has no persistent named entities — you re-select by
location each time.

## Details
`ShapeType` (`DataModelV2.py`, an `IntStrEnum`): `vertex=0`, `edge=1`, `face=2`,
`volume=3`, `void=4`. Because it is an `IntStrEnum`, the string (`'face'`), the int (`2`),
and `ShapeType.face` are interchangeable — API methods accept `types='face'`.

### Selecting (on [[StageBase]])
- **By point:** `select(p, types='face')` — the shape of that type containing `p`.
- **By box:** `select(p0, p1, types='edge', option='blue'|'green')`.
  `'blue'` = exclusive (must be fully inside), `'green'` = inclusive (touching counts) —
  mirroring the GUI's drag-direction colors.
- **All of a type:** `faces()`, `edges()`, `vertices()`, `volumes()`, or
  `get_shapes(types=[...])`.
- **Topological:** `get_sub_shapes(shapes, sub_types)`, `get_shared_edges/faces`,
  `get_shapes_by_vertices`, `get_shape_by_id`.

### Hierarchy
A face has edges and vertices; `shape.edges()`, `shape.vertices()` (and the same on a
single-element [[ShapeList]]) descend the topology.

## Code Example
```python
# INFERRED — common selection idioms
soil_face = mod.select([5, 2.5], types='face')        # face under a point
top_edge  = mod.select([5, 5],   types='edge')        # edge at a point
all_left  = mod.select([0,0], [0,5], types='vertex', option='green')  # box, inclusive
its_edges = soil_face.edges()                          # descend topology
```

## Related
- [[ShapeList]] — the return type and its methods.
- [[StageBase]] — hosts all selection methods.
- [[Build 2D Geometry]] — selection in a real build sequence.

## Notes
- Selections are **stage/model-bound** and positional — there are no stable shape names.
  Re-select after geometry edits rather than caching `Shape` objects across changes.
- Adding lines that enclose a region auto-creates the interior face; select it with
  `types='face'` at an interior point.
