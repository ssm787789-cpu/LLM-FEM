---
title: AnalysisProperties
category: api
source: RemoteStage.py
last_updated: 2026-06-19
tags: [#stage, #solver, #mesh, #consolidation, #seepage, #script, #v2]
source_count: 1
relates_to:
  - page: "[[Stage]]"
    rel: required_by
  - page: "[[StageBase]]"
    rel: required_by
  - page: "[[Analysis Types]]"
    rel: supports
  - page: "[[Remote Sync and Caching]]"
    rel: example_of
---

# AnalysisProperties

## Summary
`AnalysisProperties` is the descriptor mixin that gives [[Stage]] and [[Model]] their
**analysis settings as plain attributes**. Assign `stage.analysis_type = 'seepage'` and it
auto-syncs to the server (same cache/dirty/batch engine as materials,
[[Remote Sync and Caching]]). It also backs the bulk `set_analysis_properties(...)` call.

## Details
Defined in `RemoteStage.py` (`class AnalysisProperties(RemoteCacheMixin)`). Every field is
a `prop(...)` descriptor. Highlights by area:

### Core
- `analysis_type` — `mesh | initial_stress | load_multiplier | deformation |
  load_deformation | factor_of_safety | seepage` (see [[Analysis Types]]).
- `element_type` — `lower | upper | mixed | gauss`.
- `no_of_elements`, `mesh_option` (`minimal|coarse|medium|fine|very_fine|user`),
  `use_model_mesh`.
- `initial_stresses` — `none | automatic`.
- `reset_displacements`, `from_model`, `from_stage`, `name`, `design_approach`
  (`unity|sls|uls|als`).

### Mesh adaptivity
`mesh_adaptivity`, `adaptivity_iterations`, `adaptivity_frequency`, `start_elements`,
`adaptivity_control` (`total_disipation | shear_disipation | strain`).

### Strength reduction (factor_of_safety)
`reduce_strength_in` (`soil | plates | anchors | plates_and_anchors`),
`strength_reduction_type` (`ultimate | target`), `strength_reduction_target_fos`,
`davis_correction`.

### Load-deformation / deformation (time)
`deformation_analysis_type` (`elastoplastic | elastic`), `time_scope`
(`long_term | short_term | variable`), `deformation_time`, `deformation_no_of_steps`,
`deformation_target` (`time | consolidation_degree`), `target_consolidation_percent`,
`load_deformation_scheme/target/step_variation`, `load_deformation_u_target`,
`load_deformation_no_of_steps`, `load_multiplier_multiplier` (`load | gravity`).

### Seepage
`seepage_time_settings` (`steady_state | transient`), `seepage_time_step_variation`,
`seepage_target_time`, `seepage_no_of_steps`, `seepage_pressure` (`auto | user`),
`seepage_user_pressure`. See [[Drainage and Seepage]].

### Seismic
`seismic_accelerations`, `seismic_kx`, `seismic_ky`, `seismic_kz`.

## Code Example
```python
# INFERRED — two equivalent ways to configure a consolidation stage
# 1) attribute style (each line = one auto-sync)
stage.analysis_type = 'deformation'
stage.deformation_target = 'consolidation_degree'
stage.target_consolidation_percent = 90

# 2) batch (one round trip)
with stage.batch():
    stage.analysis_type = 'deformation'
    stage.time_scope = 'variable'
    stage.deformation_time = 100      # days

# 3) bulk call (also one round trip)
stage.set_analysis_properties(analysis_type='factor_of_safety',
                              element_type='upper', no_of_elements=2000)
```

## Related
- [[Stage]] / [[StageBase]] — where these attributes are exposed.
- [[Analysis Types]] — semantics of `analysis_type` values.
- [[Drainage and Seepage]] — seepage/consolidation settings.
- [[Remote Sync and Caching]] — why assignment syncs and how `batch()` helps.

## Notes
- Enum fields accept the **string** name (`'lower'`), the Python enum, or the int.
- `name` is special: assigning `stage.name` routes through `rename_stage`, not
  `set_analysis_properties`.
- `from_stage`/`from_model` accept a name, a UUID, or a [[Stage]]/[[Model]] object —
  resolved to a UUID automatically.
