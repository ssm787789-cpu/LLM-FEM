---
title: Output
category: api
source: v2.py, Output.py, ResultData_pb2.py
last_updated: 2026-06-19
tags: [#output, #result, #stability, #plate, #script, #v2]
source_count: 3
relates_to:
  - page: "[[Stage]]"
    rel: required_by
  - page: "[[StageBase]]"
    rel: required_by
  - page: "[[Extract Results]]"
    rel: example_of
  - page: "[[GxClient]]"
    rel: contradicts
---

# Output

## Summary
`stage.output` (a **property** in V2) is how you pull analysis results back into Python:
critical/global scalars (load multiplier, FOS, max displacement) and per-element fields
grouped by entity type (solid, plate, geogrid, connector, result points, reactions). The
data arrives as a dense mesh you index element-by-element.

## Details
`StageBase.output` (`v2.py`) returns a `StageOutput` (subclass of `StageResults` in
`Output.py`). Two flavors of result:

### Critical / global scalars
`stage.output.critical_results` → `CriticalResults` with fields like
`factor_of_safety`, `load_multiplier`, `consolidation_degree`, `time_days`,
`u_norm_max`, stress extrema (`sx_total_max`, ...), plate force extrema
(`moment_plate2d_max`, ...), connector/anchor force extrema. There is also
`stage.output.global_results.<name>` for global solution scalars (e.g. `load_multiplier`).

### Per-entity element results
`stage.output.<group>` returns a `ResultIndexer` you can `len()` and index `[i]`:

`solid`, `plate`, `geogrid`, `connector`, `nailrow`, `pilerow`, `interface`,
`resultpoint`, `plate_resultpoint`, `geogrid_resultpoint`, `connector_resultpoint`,
`nail_row_resultpoint`, `pile_row_resultpoint`, `fixed_end_anchor_resultpoint`,
`point_reaction`, `line_reaction`, `face_reaction`.

Each element `el = stage.output.solid[i]` exposes:
- `el.topology.X / .Y / .Z` and `el.topology.nodes` — geometry.
- `el.results.<field_set>.<field>.value` (+ `.unit`) — result fields, e.g.
  `el.results.collapse_mechanism.u_norm.value`, `el.results.final_forces.M_y.value`.
- `el.material.name / .color` and `el.general.material_model / .shape_id`.

Values are numpy arrays reshaped to `[definitions, element_count, element_size]`.

## Code Example
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py) — FOS + collapse + plate forces
FOS  = mod.output.critical_results.factor_of_safety
soil = mod.output.solid
wall = mod.output.plate

for i in range(len(soil)):
    x = soil[i].topology.X
    y = soil[i].topology.Y
    u = soil[i].results.collapse_mechanism.u_norm.value

for i in range(len(wall)):
    M = wall[i].results.final_forces.M_y.value
    V = wall[i].results.final_forces.V_y.value
    N = wall[i].results.final_forces.N.value
```

## Related
- [[Stage]] / [[StageBase]] — own the `output` property.
- [[Extract Results]] — workflow for common result pulls.
- [[Read Critical Results (example)]] — worked example.

## Contradictions
**V1 vs V2 access shape.** In V2, `output` is a `@property`
(`mod.output.solid`, no parentheses) — confirmed in `v2.py` and the shipped
`example_plugin.py`. In the legacy V1 [[GxClient]], `output()` is a **method**
(`gx.output()` returns an `Output.Output`). Same word, different call convention across
generations. Do not call `mod.output()` on a V2 [[Stage]]/[[Model]], and do not access
`gx.output.solid` on a V1 client. Preserved, not resolved — pick the surface that matches
your client. See [[V1 vs V2 API]].

## Notes
- Result field sets available depend on the `analysis_type`: e.g. `collapse_mechanism`
  for limit analysis, `final_forces` for structural elements, displacement/stress fields
  for deformation analyses.
- Guard loops with `if len(group):` — a group property is `None`/empty when that entity
  type isn't present in the model.
