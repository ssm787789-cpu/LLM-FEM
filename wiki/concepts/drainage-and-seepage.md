---
title: Drainage and Seepage
category: concept
source: Materials/MohrCoulomb.py, RemoteStage.py, ContractV2_enum.py
last_updated: 2026-06-19
tags: [#seepage, #consolidation, #material, #boundary, #script]
source_count: 3
relates_to:
  - page: "[[MohrCoulomb (material)]]"
    rel: supports
  - page: "[[AnalysisProperties]]"
    rel: supports
  - page: "[[FeatureAPI]]"
    rel: supports
  - page: "[[Analysis Types]]"
    rel: supports
---

# Drainage and Seepage

## Summary
Pore water shows up in three places in a script: the **material's** drainage/hydraulic
parameters, the **stage's** seepage/consolidation settings, and the **hydraulic boundary
conditions** (water tables, heads, fixed pressures). This page ties the three together so
you know which knob lives where.

## Details

### On the material ([[MohrCoulomb (material)]] and siblings)
- `drainage` ∈ `drained_undrained | always_drained | non_porous`.
- `hydraulic_model` ∈ `basic | van_genuchten`;
  `hydraulic_conductivity_option` ∈ `isotropic | anisotropic`.
- Conductivities `Kx, Ky, Kz` (or `K`), void ratio `e`, van Genuchten `alpha`, `n`,
  saturations `Sr`, `Ss`.
- `excess_pore_pressure`, `cavitation_cutoff`, `pcav`.

### On the stage ([[AnalysisProperties]])
- `analysis_type = 'seepage'` for a flow/pressure field;
  `seepage_time_settings` ∈ `steady_state | transient`, `seepage_time_step_variation`,
  `seepage_target_time`, `seepage_no_of_steps`.
- `seepage_pressure` ∈ `auto | user`, `seepage_user_pressure`.
- For **consolidation**: `analysis_type='deformation'` with
  `deformation_target='consolidation_degree'` and `target_consolidation_percent`, plus
  `time_scope` (`long_term | short_term | variable`).

### Hydraulic boundary conditions ([[FeatureAPI]])
`set_water_table`, `set_fixed_head`, `set_fixed_pressure`, `set_fixed_excess_pressure`,
`set_no_flow`, `set_seepage_face`.

## Code Example
```python
# INFERRED — steady-state seepage stage with a water table
clay = prj.MohrCoulomb(name='Clay', phi=24, c=5,
                       drainage='drained_undrained',
                       Kx=1e-8, Ky=1e-8)
wt = mod.select([0,3], [20,3], types='edge', option='green')
mod.set_water_table(wt)
s = mod.create_stage('seepage')
s.set_analysis_properties(analysis_type='seepage', seepage_time_settings='steady_state')
prj.run_analysis()
```

## Related
- [[MohrCoulomb (material)]] — material hydraulic parameters.
- [[AnalysisProperties]] — stage seepage/consolidation settings.
- [[FeatureAPI]] — hydraulic BCs.
- [[Analysis Types]] — `seepage` vs `deformation` (consolidation).

## Notes
- `always_drained` ignores excess pore pressure; `drained_undrained` lets the analysis
  decide based on time scope; `non_porous` disables pore water.
- Short-term ≈ undrained, long-term ≈ drained — set `time_scope` accordingly for
  deformation/consolidation stages.
