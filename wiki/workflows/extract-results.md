---
title: Extract Results
category: workflow
source: v2.py, Output.py, plugins/Example_Plugin/example_plugin.py
last_updated: 2026-06-19
tags: [#output, #result, #script, #v2]
source_count: 3
relates_to:
  - page: "[[Output]]"
    rel: example_of
  - page: "[[Stage]]"
    rel: supports
  - page: "[[Run an Analysis]]"
    rel: required_by
  - page: "[[Read Critical Results (example)]]"
    rel: example_of
---

# Extract Results

## Summary
After `run_analysis()`, pull numbers from `stage.output` ([[Output]]): scalar
critical/global results, or per-element fields by looping the entity groups.

## Details
- **Scalars:** `stage.output.critical_results.<field>` (e.g. `factor_of_safety`,
  `load_multiplier`, `consolidation_degree`, `u_norm_max`) and
  `stage.output.global_results.<field>`.
- **Per-element:** index a group (`solid`, `plate`, `geogrid`, `connector`,
  `*_resultpoint`, `*_reaction`, ...). Each element exposes `.topology` (X/Y/Z, nodes),
  `.results.<field_set>.<field>.value` (+ `.unit`), and `.material`.

## Steps
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py) — scalar + per-element fields
FOS  = mod.output.critical_results.factor_of_safety
soil = mod.output.solid
wall = mod.output.plate

Xs, Ys, Us = [], [], []
for i in range(len(soil)):
    Xs.append(soil[i].topology.X)
    Ys.append(soil[i].topology.Y)
    Us.append(soil[i].results.collapse_mechanism.u_norm.value)

for i in range(len(wall)):
    M = wall[i].results.final_forces.M_y.value
    V = wall[i].results.final_forces.V_y.value
    N = wall[i].results.final_forces.N.value
```

## Related
- [[Output]] — the full result object model and field-set notes.
- [[Run an Analysis]] — produces the results.
- [[Read Critical Results (example)]] — focused scalar-extraction example.
- [[Stage]] — owns `output`.

## Notes
- `stage.output` is a **property** in V2 (no parentheses). V1 uses `gx.output()`
  (method) — see [[Output]] Contradictions.
- Guard groups with `if len(group):` — a group is empty/`None` when that entity type is
  absent.
- Available `field_set`s depend on `analysis_type` (e.g. `collapse_mechanism` for limit
  analysis, `final_forces` for structural elements).
