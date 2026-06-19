---
title: Staged Construction
category: concept
source: v2.py, RemoteStage.py
last_updated: 2026-06-19
tags: [#stage, #phase, #excavation, #consolidation, #script]
source_count: 2
relates_to:
  - page: "[[Stage]]"
    rel: supports
  - page: "[[Model]]"
    rel: supports
  - page: "[[Analysis Types]]"
    rel: supports
  - page: "[[FeatureAPI]]"
    rel: supports
---

# Staged Construction

## Summary
Real geotechnical analyses are sequences of phases — establish initial stress, excavate,
install a wall, dewater, load. In OptumGX each phase is a [[Stage]] that **continues from
a previous stage** (`from_stage`) and toggles which features are active. This is how
excavation, construction, and consolidation sequences are scripted.

## Details
- A [[Model]] owns an ordered set of [[Stage]] objects: `mod.create_stage(name)`,
  `mod.get_stage()`, `mod.get_current_stage()`, `mod.set_current_stage(...)`.
- State inheritance: `stage.from_stage = previous_stage` (or `set_from_stage(...)`).
  Stresses/displacements carry forward from the referenced stage. `from_stage` accepts a
  name, UUID, or [[Stage]] object.
- Each stage sets its own `analysis_type` and settings via [[AnalysisProperties]].
- Feature activation: features are created on the [[Model]] or [[Stage]]; the
  `StageFeatureAPI` lets a stage toggle which are active (e.g. excavate by deactivating a
  soil cluster, install a wall by activating a plate).
- Only stages flagged with `set_run_flag(RunFlag.run)` are solved by
  `prj.run_analysis()`.

## Code Example
```python
# INFERRED — initial stress -> excavation -> FOS, chained
s0 = mod.create_stage('initial')
s0.set_analysis_properties(analysis_type='initial_stress', initial_stresses='automatic')

s1 = mod.create_stage('excavate')
s1.from_stage = s0
s1.analysis_type = 'deformation'
# (deactivate the excavated cluster via the relevant feature toggle here)

s2 = mod.create_stage('FOS')
s2.from_stage = s1
s2.set_analysis_properties(analysis_type='factor_of_safety', element_type='mixed')

prj.run_analysis()
print('FOS =', s2.output.critical_results.factor_of_safety)
```

## Related
- [[Stage]] — the phase object and `from_stage`.
- [[Model]] — owns the stage sequence and the geometry stages share.
- [[Analysis Types]] — what each phase computes.
- [[FeatureAPI]] — features toggled per stage.

## Notes
- Geometry is shared from the [[Model]]; stages don't rebuild geometry, they change
  *which* of it is active and *how* it is analyzed.
- A typical first stage is `initial_stress`; chain everything else from it.
