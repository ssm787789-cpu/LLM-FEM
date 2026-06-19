---
title: Apply Loads and Boundary Conditions
category: workflow
source: RemoteFeatures/FeatureAPI.py, GxClient.py
last_updated: 2026-06-19
tags: [#load, #boundary, #selection, #script, #v2]
source_count: 2
relates_to:
  - page: "[[FeatureAPI]]"
    rel: example_of
  - page: "[[ShapeList]]"
    rel: supports
  - page: "[[Csys]]"
    rel: supports
  - page: "[[Assign Materials]]"
    rel: supports
---

# Apply Loads and Boundary Conditions

## Summary
Loads and supports are [[FeatureAPI|features]] applied to a [[ShapeList]]. Select the
target shapes, call the matching `set_*`, and (optionally) keep the returned feature
object to edit later. Most models also need `set_standard_fixities()` to restrain the
domain boundary.

## Details
- **Supports / fixities:** `set_standard_fixities()` (auto-restrain outer boundary),
  `set_support(sel, type='normal'|'tangential'|'full')`, `set_platebc(...)`,
  `set_point_bc(...)`.
- **Loads:** `set_point_load`, `set_line_load`, `set_surface_load`, `set_body_load`,
  `set_point_moment`, `set_line_moment`, `set_six_dof_load`.
- **Direction / option:** `direction='x'|'y'|'z'`, `option='multiplier'|'fixed'|
  'dead_load'|'live_load'`, `load_category` for safety classes, `csys=` a [[Csys]] for a
  non-global frame.
- **Magnitude:** a constant, or a `ParameterMap`/`Gradient` for varying loads
  ([[Field Values (ParameterMap, Profile, Gradient)]]).

## Steps
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py) — standard fixities
mod.set_standard_fixities()
```
```python
# INFERRED — a downward line load and a normal support
top  = mod.select([0,5],[10,5], types='edge', option='green')
mod.set_line_load(top, p=-25, direction='z', option='multiplier')

base = mod.select([0,0],[10,0], types='edge', option='green')
mod.set_support(base, type='full')
```
```python
# INFERRED — edit a feature after creation (auto-syncs)
ld = mod.set_point_load(mod.select([5,5], types='vertex'), p=-100, direction='z')
# ld.p = -150   # adjust later; pushes to server
```

## Related
- [[FeatureAPI]] — the full setter/getter list.
- [[ShapeList]] — selection handles.
- [[Csys]] — non-global load/support frames.
- [[Run an Analysis]] — next step.

## Notes
- `option='multiplier'` makes the load scale with the analysis load multiplier (used in
  `load_multiplier` limit analysis); `'fixed'`/`'dead_load'` are non-scaling.
- For staged models, which features are *active* is per-stage — see
  [[Staged Construction]].
