---
title: Read Critical Results (example)
category: example
source: Output.py, plugins/Example_Plugin/example_plugin.py
last_updated: 2026-06-19
tags: [#output, #result, #script, #v2]
source_count: 2
relates_to:
  - page: "[[Output]]"
    rel: example_of
  - page: "[[Extract Results]]"
    rel: example_of
---

# Read Critical Results (example)

## Summary
A focused recipe for pulling scalar summary numbers out of a solved stage — the fields on
`CriticalResults` — and iterating max/min displacements and structural force extrema.

## Code Example
```python
# INFERRED — scalar critical-results dump (field names VERBATIM from Output.py CriticalResults)
cr = stage.output.critical_results

# Limit / safety
print('FOS                 :', cr.factor_of_safety)
print('load multiplier     :', cr.load_multiplier)
print('consolidation degree:', cr.consolidation_degree)
print('time (days)         :', cr.time_days)

# Displacement extrema
print('|u| max             :', cr.u_norm_max)
print('ux max / uy max     :', cr.u_x_max, cr.u_y_max)
print('solid |u| max       :', cr.u_solid_norm_max)

# Stress extrema (effective)
print('sx_eff min/max      :', cr.sx_effective_min, cr.sx_effective_max)

# Structural force extrema
print('plate M (2D) min/max:', cr.moment_plate2d_min, cr.moment_plate2d_max)
print('connector force max :', cr.connector_force_max)
print('anchor force max    :', cr.fixed_end_anchor_force_max)
```

## Related
- [[Output]] — the full `CriticalResults` field list and per-element groups.
- [[Extract Results]] — the broader extraction workflow.

## Notes
- `CriticalResults` is populated dynamically from the server's `StepCriticalData`; a field
  is `None` if that quantity wasn't produced by the stage's `analysis_type`. Guard reads.
- The field names above are copied verbatim from `Output.py`'s `CriticalResults` class
  annotations; the surrounding print loop is INFERRED.
- For per-element fields (not scalars) loop the groups — see [[Extract Results]].
