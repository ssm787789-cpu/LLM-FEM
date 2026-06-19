---
title: Project
category: api
source: v2.py, Materials/RemoteMaterialAPI.py
last_updated: 2026-06-19
tags: [#script, #automation, #v2, #material]
source_count: 2
relates_to:
  - page: "[[GX]]"
    rel: required_by
  - page: "[[Model]]"
    rel: required_by
  - page: "[[RemoteMaterialAPI]]"
    rel: extends
  - page: "[[Csys]]"
    rel: supports
---

# Project

## Summary
`Project` is the container for models, materials, and coordinate systems in the **V2**
API, and the object you call to run analyses. You get one from [[GX]]
(`create_project` / `get_current_project` / `open_project`). Because `Project` inherits
[[RemoteMaterialAPI]], **all material factory methods live here** (e.g.
`prj.MohrCoulomb(...)`).

## Details
Defined in `v2.py` (`class Project(RemoteMaterialAPI)`).

### Models
| Method | Returns | Purpose |
|--------|---------|---------|
| `create_model(name=None, model_type=None)` | [[Model]] | Create a model. `model_type` ∈ `ModelType.{plane_strain, axisymmetric, three_dimensional}`. |
| `get_model(name=None)` | [[Model]] / list | Get model(s) by name. |
| `get_current_model()` | [[Model]] | Model currently active in the GUI. |
| `set_current_model(name_or_model)` | — | Make a model active. |
| `get_current_stage()` | [[Stage]] | Active stage of the current model. |
| `get_models_and_stages()` | list | All models and stages. |

### Materials
Inherited from [[RemoteMaterialAPI]]: per-type constructors such as
`MohrCoulomb(...)`, `LinearElastic(...)` (aliased `ElasticSolid`), `GeneralPlate(...)`,
`Tresca(...)`, plus `get_material(name=None)` and category getters. Each constructor
**creates the material on the server immediately** and returns a live remote object.

### Coordinate systems
`create_csys_2d(origo, direction_i, name)`, `create_csys_3d(origo, direction_i,
direction_j, name)`, `get_csys(name=None)` → [[Csys]].

### Running & design approaches
| Method | Purpose |
|--------|---------|
| `run_analysis()` | Solve all run-flagged stages/models. |
| `set_design_approach_parameters(name, parameters)` | Configure a design approach (e.g. partial factors). |
| `get_design_approach(name)` | Read a design approach. |
| `get_python_code()` | Return Python that reconstructs the current project (great for learning the API). |
| `get_current_result_set()` | The result field currently plotted in the GUI → `PlottedResult`. |

## Code Example
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py)
gx = GX()
prj = gx.create_project('Cantilever Wall')
mod = prj.create_model('FOS')
Soil = prj.MohrCoulomb(name='Soil', c=c, phi=phi, gamma_dry=gamma, color=rgb(180,180,90))
Wall = prj.GeneralPlate(name='Wall', m_px=mp, color='blue')
# ...
prj.run_analysis()
```

## Related
- [[GX]] — produces `Project` instances.
- [[Model]] — created by `create_model`; holds geometry.
- [[RemoteMaterialAPI]] — the material factory mixed into `Project`.
- [[Run an Analysis]] — uses `prj.run_analysis()`.

## Notes
- `prj.get_python_code()` is the fastest way to discover correct calls: build something
  in the GUI, then print the generated script.
- Materials are project-scoped; create them once and assign to shapes across models.
