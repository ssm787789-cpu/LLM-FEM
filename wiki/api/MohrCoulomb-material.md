---
title: MohrCoulomb (material)
category: api
source: Materials/MohrCoulomb.py, Materials/RemoteMaterialAPI.py
last_updated: 2026-06-19
tags: [#material, #script, #v2]
source_count: 2
relates_to:
  - page: "[[RemoteMaterial]]"
    rel: extends
  - page: "[[RemoteMaterialAPI]]"
    rel: required_by
  - page: "[[Material Models]]"
    rel: example_of
  - page: "[[Field Values (ParameterMap, Profile, Gradient)]]"
    rel: supports
---

# MohrCoulomb (material)

## Summary
The `MohrCoulomb` material is the workhorse elasto-plastic soil model in OptumGX, and a
representative example of how every [[RemoteMaterial]] subclass is structured: a set of
typed, auto-syncing properties grouped into elastic, strength, hydraulic, and advanced
blocks. Create it with `prj.MohrCoulomb(...)` ([[RemoteMaterialAPI]]).

## Details
Defined in `Materials/MohrCoulomb.py`. Properties (all assignable; most accept
`float | ParameterMap | Profile | Gradient` — see
[[Field Values (ParameterMap, Profile, Gradient)]]):

| Group | Properties |
|-------|------------|
| Identity | `name`, `color` |
| Unit weight | `gamma_dry`, `gamma_sat` |
| Elastic | `E`, `nu`, `K`, `G`, `parameter_set` (`'A'`=E,nu · `'B'`=K,nu · `'C'`=K,G) |
| Strength | `phi`, `c`, `psi`, `reducible_strength` |
| Tension cutoff | `tension_cutoff`, `sigma_t` |
| Flow rule | `flow_rule`, `dilation_cap`, `ev_cr`, `es_cr` |
| Compression cap | `compression_cap`, `compression_strength` |
| Softening | `softening`, `phi_residual`, `e_cr1`, `e_cr2` |
| Hydraulic | `drainage`, `hydraulic_model`, `hydraulic_conductivity_option`, `Kx`,`Ky`,`Kz`, `excess_pore_pressure`, `K0`, `K0_type`, `cavitation_cutoff`, `pcav`, `e`, `alpha`, `n`, `Sr`, `Ss` |

The proto oneof field name is `mohr_coulomb`; nested paths are declared in
`_PROPERTY_PATHS` (e.g. `'phi' -> ('mohr_coulomb','phi')`, `'E' -> ('elastic','E')`,
`'drainage' -> ('hydraulic','drainage')`).

### Drainage / hydraulic enums (string-valued)
`drainage` ∈ `drained_undrained | always_drained | non_porous`;
`hydraulic_model` ∈ `basic | van_genuchten`;
`hydraulic_conductivity_option` ∈ `isotropic | anisotropic`. See [[Drainage and Seepage]].

## Code Example
```python
# INFERRED — drained sand with a depth gradient on stiffness
sand = prj.MohrCoulomb(
    name='Sand', phi=34, c=0.1, psi=4,
    gamma_dry=17, gamma_sat=20,
    E=Gradient(zref=0, zgrad=2000, value=20000),   # E grows with depth
    nu=0.3, drainage='always_drained',
)
sand.tension_cutoff = True
sand.sigma_t = 0.0
```

## Related
- [[RemoteMaterial]] — base class (sync/batch/cache behavior).
- [[RemoteMaterialAPI]] — the `prj.MohrCoulomb(...)` constructor and full kwarg list.
- [[Material Models]] — sibling models (Tresca, HardeningSoil, GeneralPlate, ...).
- [[Field Values (ParameterMap, Profile, Gradient)]] — `E=Gradient(...)` etc.
- [[Drainage and Seepage]] — meaning of `drainage`/hydraulic params.

## Notes
- `MohrCoulombEngineering` is a variant taking engineering strength inputs (`cu`,
  Davis-corrected `c/phi` per 2D/3D) and pairs with the `davis_correction` stage flag.
- `reducible_strength=True` makes this material participate in factor-of-safety strength
  reduction (see [[Analysis Types]] / `factor_of_safety`).
