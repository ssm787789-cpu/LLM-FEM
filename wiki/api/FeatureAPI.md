---
title: FeatureAPI
category: api
source: RemoteFeatures/FeatureAPI.py
last_updated: 2026-06-19
tags: [#load, #boundary, #plate, #pile, #anchor, #seepage, #script, #v2]
source_count: 1
relates_to:
  - page: "[[Model]]"
    rel: required_by
  - page: "[[Stage]]"
    rel: required_by
  - page: "[[ShapeList]]"
    rel: required_by
  - page: "[[Apply Loads and Boundary Conditions]]"
    rel: example_of
---

# FeatureAPI

## Summary
`FeatureAPI` is the mixin that adds **loads, boundary conditions, and structural
elements** to [[Model]] (`ModelFeatureAPI`) and [[Stage]] (`StageFeatureAPI`). Every
`set_*` method takes a [[ShapeList]] plus parameters, creates a remote feature, and
returns a live feature object you can edit later. Every `get_*` method reads existing
features back.

## Details
Defined in `RemoteFeatures/FeatureAPI.py`; each feature type has its own collection class
in `RemoteFeatures/<Name>.py`.

### Loads
| Setter | Feature | Key params |
|--------|---------|-----------|
| `set_point_load(shapes, p, direction, option, csys, load_type, load_category)` | `PointLoads` | `p` float |
| `set_line_load(...)` | `LineLoads` | `p` may be `float\|ParameterMap\|Gradient`; `load_variation` |
| `set_surface_load(...)` | `SurfaceLoads` | as line load |
| `set_body_load` / volume load | `VolumeLoads` | |
| `set_point_moment`, `set_line_moment`, `set_six_dof_load` | moments / 6-DOF | |

Shared load enums: `direction` ∈ `x|y|z` (`LoadDirection`), `option` ∈
`multiplier|fixed|dead_load|live_load` (`LoadOption`), `load_category` ∈
`unfavourable|favourable|dead_load|live_load|user_1|user_2` (`SafetyType`),
`load_variation` ∈ `constant|linear`, `csys` a [[Csys]].

### Boundary conditions & supports
`set_support(shapes, type=normal|tangential|full)`, `set_platebc(...)`,
`set_point_bc(...)`, `set_standard_fixities()` (model-level), plus
`set_prestress`, `set_reaction_relaxation`.

### Structural elements
`set_plate(shapes, material, strength_reduction_factor, tension_cutoff,
compression_cutoff)`, `set_geogrid`, `set_nailrow`, `set_pilerow`,
`set_fixed_end_anchor`, `set_connector` (`add_connector`), `set_shear_joint`,
`set_solid(shapes, material)`.

### Seepage / hydraulic BCs
`set_water_table`, `set_fixed_head`, `set_fixed_pressure`, `set_fixed_excess_pressure`,
`set_no_flow`, `set_seepage_face`.

### Mesh & results
`set_mesh_size`, `set_mesh_fan`, `set_result_point`, `set_extrude_to_3d` (2D→3D control).

### Reading & toggling
`get_<feature>(shapes)` returns the matching feature collection or `None`.
`StageFeatureAPI` adds staged activation (`toggle_features`); `ModelFeatureAPI` adds
`remove_features`, `set_standard_fixities`.

## Code Example
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py) — plate + solid + fixities
sel = mod.select([0,0], types='face')
mod.set_solid(sel, Soil)
sel = mod.select([0,-D/2], types='edge')
mod.set_plate(sel, Wall, strength_reduction_factor=r)
mod.set_standard_fixities()
```
```python
# INFERRED — distributed surface load and a normal support
top = mod.select([5,5], types='edge')
mod.set_line_load(top, p=-20, direction='z', option='multiplier')
side = mod.select([0,2], types='edge')
mod.set_support(side, type='normal')
```

## Related
- [[Model]] / [[Stage]] — host the feature methods.
- [[ShapeList]] — every feature is applied to a selection.
- [[Apply Loads and Boundary Conditions]] — workflow.
- [[Csys]] — load direction reference frames.

## Notes
- Setters create immediately and return an editable feature object; assign to its
  properties (auto-syncs) to tweak after creation.
- A single `set_*` may pass a single `Shape` or a `ShapeList` — both are accepted.
- Staged construction toggles which features are *active* per [[Stage]]; see
  [[Staged Construction]].
