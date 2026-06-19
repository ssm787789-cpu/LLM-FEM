---
title: Assign Materials
category: workflow
source: v2.py, Materials/RemoteMaterialAPI.py, plugins/Example_Plugin/example_plugin.py
last_updated: 2026-06-19
tags: [#material, #selection, #script, #v2]
source_count: 3
relates_to:
  - page: "[[RemoteMaterialAPI]]"
    rel: example_of
  - page: "[[Material Models]]"
    rel: supports
  - page: "[[Build 2D Geometry]]"
    rel: required_by
  - page: "[[FeatureAPI]]"
    rel: supports
---

# Assign Materials

## Summary
Create a material on the [[Project]], select the shapes it applies to, then bind them with
the matching `set_*` method. Material **type must match the assignment method** (solids →
`set_solid`, plates → `set_plate`, etc.).

## Details
1. Create with `prj.<Model>(...)` ([[RemoteMaterialAPI]]).
2. Select target shapes ([[Shapes and Selection]]).
3. Assign:

| Material kind | Assignment | Applies to |
|---------------|-----------|-----------|
| Solid (soil/rock/elastic) | `mod.set_solid(sel, mat)` | faces (2D) / volumes (3D) |
| Plate/shell | `mod.set_plate(sel, mat, strength_reduction_factor=...)` | edges (2D) / faces (3D) |
| Geogrid | `mod.set_geogrid(sel, mat)` | edges/faces |
| Nail row | `mod.set_nailrow(sel, mat)` | edges |
| Pile row | `mod.set_pilerow(sel, mat)` | edges |
| Connector/anchor | `mod.add_connector(p0, p1, mat)` | a line between points |
| Shear joint | `mod.set_shear_joint(sel, mat)` | shared edges/faces |

## Steps
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py)
Soil = prj.MohrCoulomb(name='Soil', c=c, phi=phi, gamma_dry=gamma, color=rgb(180,180,90))
Wall = prj.GeneralPlate(name='Wall', m_px=mp, color='blue')

sel = mod.select([0,0], types='face')
mod.set_solid(sel, Soil)

sel = mod.select([0,-D/2], types='edge')
mod.set_plate(sel, Wall, strength_reduction_factor=r)
sel = mod.select([0,H/2], types='edge')
mod.set_plate(sel, Wall, strength_reduction_factor=r)
```

## Related
- [[RemoteMaterialAPI]] — material constructors and getters.
- [[Material Models]] — picking the right model and its assignment method.
- [[Build 2D Geometry]] — produces the shapes to assign to.
- [[FeatureAPI]] — `set_solid`/`set_plate`/... live here.

## Notes
- Materials are project-scoped: define once, reuse across faces/models.
- `set_plate(..., strength_reduction_factor=r)` scales plate strength (e.g. interface
  roughness). `tension_cutoff`/`compression_cutoff` flags are also available.
- Mismatched type↔method (e.g. `set_plate` with a solid material) errors server-side.
