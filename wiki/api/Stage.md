---
title: Stage
category: api
source: v2.py, RemoteStage.py
last_updated: 2026-06-19
tags: [#stage, #phase, #solver, #script, #v2, #excavation]
source_count: 2
relates_to:
  - page: "[[Model]]"
    rel: required_by
  - page: "[[StageBase]]"
    rel: extends
  - page: "[[AnalysisProperties]]"
    rel: extends
  - page: "[[Staged Construction]]"
    rel: example_of
---

# Stage

## Summary
A `Stage` is one analysis **phase** of a [[Model]] — e.g. an initial-stress stage, an
excavation step, or a factor-of-safety check. Stages inherit the model's geometry and
differ in their **analysis settings** ([[AnalysisProperties]]), active features, and the
stage they continue from (`from_stage`). This is the unit of staged construction.

## Details
Defined in `v2.py` as `class Stage(StageBase, StageFeatureAPI)`. It inherits:
- geometry-agnostic methods from [[StageBase]] (select, get_shapes, `output`,
  `set_solver_settings`, `set_run_flag`),
- descriptor-based analysis settings from [[AnalysisProperties]] (assign directly,
  e.g. `stage.element_type = 'lower'`),
- staged feature activation from [[FeatureAPI]] (`StageFeatureAPI` variant).

### Key members
| Member | Purpose |
|--------|---------|
| `model` | The parent [[Model]]. |
| `from_stage` (get/set), `set_from_stage(s)` | The stage this one continues from (state inheritance). |
| `create_stage(name)` | Create the next stage after this one. |
| `output` (property) | Analysis results → see [[Output]]. **Property, not a method** in V2. |
| `set_run_flag(RunFlag.run)` / `get_run_flag()` | Include/exclude from `run_analysis`. |
| `set_solver_settings(...)` | Solver type/method (see [[StageBase]]). |

### Analysis settings (via [[AnalysisProperties]])
Assign as attributes — each write **auto-syncs** to the server:
```python
stage.analysis_type = 'load_multiplier'   # mesh | initial_stress | load_multiplier |
                                          # deformation | load_deformation |
                                          # factor_of_safety | seepage
stage.element_type = 'lower'              # lower | upper | mixed | gauss
stage.no_of_elements = 2000
stage.mesh_adaptivity = True
```
Use `with stage.batch():` to push several settings in one round trip.

## Code Example
```python
# VERBATIM (v2.py StageBase.output docstring) — lower-bound load multiplier stage
stage_lower = model2d.create_stage('lower bound')
stage_lower.set_analysis_properties(
    analysis_type='load_multiplier',
    element_type='lower',
    no_of_elements=1000,
)
prj.run_analysis()
print(stage_lower.output.global_results.load_multiplier)
```

## Related
- [[Model]] — owns geometry and creates stages.
- [[StageBase]] — shared base (also parent of `Model`).
- [[AnalysisProperties]] — the full list of assignable stage settings.
- [[Output]] — `stage.output` results.
- [[Staged Construction]] — how to chain stages with `from_stage`.

## Notes
- Two ways to set analysis settings: bulk `set_analysis_properties(...)` (one call) or
  attribute assignment (`stage.x = ...`, auto-syncs each). Both ultimately call the same
  gRPC `set_analysis_properties`.
- `output` is a `@property` in V2 (`stage.output`); the legacy V1 [[GxClient]] exposes
  `gx.output()` as a **method**. Don't mix the styles. See [[Output]] Contradictions.
