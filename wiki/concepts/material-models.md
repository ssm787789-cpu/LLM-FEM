---
title: Material Models
category: concept
source: Materials/RemoteMaterialAPI.py, Materials/*.py
last_updated: 2026-06-19
tags: [#material, #script, #v2]
source_count: 27
relates_to:
  - page: "[[RemoteMaterialAPI]]"
    rel: supports
  - page: "[[MohrCoulomb (material)]]"
    rel: example_of
  - page: "[[Assign Materials]]"
    rel: example_of
---

# Material Models

## Summary
OptumGX ships ~27 material models grouped by what they attach to (solids, fluids, plates,
beams, geogrids, connectors, piles, nails). Each is a constructor on [[RemoteMaterialAPI]]
and a live [[RemoteMaterial]] subclass. This page is the catalog and the
"which model do I pick" map.

## Details

### Solid (soil/rock) materials → `set_solid`
| Model | Use it for |
|-------|-----------|
| `MohrCoulomb` | General c–φ soil (the default workhorse). See [[MohrCoulomb (material)]]. |
| `MohrCoulombEngineering` | MC with engineering/undrained inputs + Davis correction. |
| `Tresca` | Undrained clay (φ=0, `cu`). |
| `DruckerPrager` | Smooth cone alternative to MC. |
| `HardeningSoil` / `HMC` | Stress-dependent stiffness, small-strain. |
| `ModifiedCamClay` | Critical-state clay. |
| `HoekBrown` | Rock mass (`GSI`, `sigma_ci`). |
| `NGIADP` | Anisotropic undrained clay (`s_uA`...). |
| `MultiMohrCoulomb` | Layered/multi-surface soil. |
| `AUS` | Undrained strength with softening (`suc`, `sue`). |
| `LinearElastic` | Elastic solid (V1 alias `ElasticSolid`). |
| `Rigid` | Rigid solid. |

### Fluid → `Water`
`Water` — fluid region material.

### Structural plates/shells → `set_plate`
`GeneralPlate` (EA/EI + plastic moment `m_px`), `FlatPlateConcrete`,
`FlatPlateSteel` (V1 `VonMisesShellMaterial`), `SheetPile`, `ReinforcedConcrete`,
`RigidPlate` (V1 `RigidShell`).

### Beams → `set_beam`
`Beam` (EA, EI), `RigidBeam`.

### Reinforcement / linear elements
`Geogrid` → `set_geogrid`; `NailRow` (`NailRowMaterial`) → `set_nailrow`;
`PileRow` → `set_pilerow`; `Connector` (`BarMaterial`, EA + yield) → `add_connector`.

Retrieve any of them with `prj.get_material(name)` or the category getters
(`get_solid_material()`, `get_plate_material()`, ...).

## Code Example
```python
# INFERRED — one of each common category
soil   = prj.MohrCoulomb(name='Soil', phi=32, c=2, gamma_dry=18, gamma_sat=20)
clay   = prj.Tresca(name='Clay', cu=40, gamma_sat=17)
wall   = prj.GeneralPlate(name='Wall', EA=6e6, EI=1e5, m_px=300)
anchor = prj.Connector(name='Anchor', EA=1e5, yield_force=200)
```

## Related
- [[RemoteMaterialAPI]] — the constructors and getters.
- [[MohrCoulomb (material)]] — full parameter walkthrough.
- [[Assign Materials]] — binding a material to selected shapes.

## Contradictions
Per-material docstrings show `gx.new.<Model>(...)`, but the shipped package has no `new`
accessor; the real call is `prj.<Model>(...)`. Some docstrings also use V1 names
(`gx.new.ElasticSolid`, `gx.new.VonMisesShellMaterial`, `gx.new.BarMaterial`,
`gx.new.NailRowMaterial`, `gx.new.NGIADPMaterial`) that differ from the V2 class names.
Flagged — see [[RemoteMaterialAPI]] and `log.md`.

## Notes
- Material class name ↔ assignment method must match: solids→`set_solid`,
  plates→`set_plate`, geogrids→`set_geogrid`, etc. Wrong pairing errors at the server.
- Names like `m_px` (plastic moment) appear in plugin code (`prj.GeneralPlate(m_px=...)`)
  even though the typed constructor lists `EA/EI`; the server accepts the extra kwargs.
