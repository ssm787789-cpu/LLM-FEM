---
title: Analysis Types
category: concept
source: DataModelV2.py, RemoteStage.py
last_updated: 2026-06-19
tags: [#stage, #solver, #stability, #consolidation, #seepage, #script]
source_count: 2
relates_to:
  - page: "[[AnalysisProperties]]"
    rel: supports
  - page: "[[Stage]]"
    rel: supports
  - page: "[[Output]]"
    rel: supports
  - page: "[[Run an Analysis]]"
    rel: example_of
---

# Analysis Types

## Summary
The `analysis_type` of a [[Stage]] (or [[Model]]) selects what OptumGX computes. It drives
which settings matter, which solver runs, and which result fields appear in [[Output]].
There are seven, set as a string on [[AnalysisProperties]].

## Details
`AnalysisType` (`DataModelV2.py`): values `mesh`, `initial_stress`, `load_multiplier`,
`deformation`, `load_deformation`, `factor_of_safety`, `seepage`.

| `analysis_type` | Computes | Key extra settings | Main result |
|-----------------|----------|--------------------|-------------|
| `mesh` | Mesh only | `element_type`, `no_of_elements`, adaptivity | mesh |
| `initial_stress` | In-situ stress (K0/gravity) | `initial_stresses` | stress field |
| `load_multiplier` | Limit load (collapse) | `element_type` (lower/upper), `load_multiplier_multiplier` | `load_multiplier`, `collapse_mechanism` |
| `factor_of_safety` | Strength-reduction FOS | `reduce_strength_in`, `strength_reduction_type`, `strength_reduction_target_fos`, `davis_correction` | `factor_of_safety`, `collapse_mechanism` |
| `deformation` | Elasto-plastic deformation / consolidation | `deformation_analysis_type`, `time_scope`, time/consolidation targets | displacements, stresses |
| `load_deformation` | Load-stepped deformation | `load_deformation_scheme/target`, steps | load–displacement path |
| `seepage` | Pore-pressure / flow field | `seepage_time_settings`, `seepage_pressure` | heads, flows |

Limit analyses (`load_multiplier`, `factor_of_safety`) hinge on `element_type`:
`lower` (safe/lower-bound) vs `upper` (unsafe/upper-bound) bracket the true answer;
`mixed`/`gauss` are practical compromises.

## Code Example
```python
# INFERRED — bracket a slope FOS with lower & upper bound stages
for et in ('lower', 'upper'):
    s = mod.create_stage(f'FOS {et}')
    s.set_analysis_properties(analysis_type='factor_of_safety',
                              element_type=et, no_of_elements=2000)
prj.run_analysis()
for s in mod.get_stage():
    print(s.name, s.output.critical_results.factor_of_safety)
```

## Related
- [[AnalysisProperties]] — every setting referenced above.
- [[Stage]] — where `analysis_type` is set.
- [[Output]] — result fields per analysis type.
- [[Run an Analysis]] — executing and reading back.

## Notes
- `initial_stress` is typically the first stage; later stages chain from it via
  `from_stage` ([[Staged Construction]]).
- For consolidation, use `deformation` with `deformation_target='consolidation_degree'`
  and `target_consolidation_percent` (see [[Drainage and Seepage]]).
