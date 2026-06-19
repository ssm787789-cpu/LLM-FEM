---
title: Cantilever Wall FOS (example)
category: example
source: plugins/Example_Plugin/example_plugin.py
last_updated: 2026-06-19
tags: [#stability, #plate, #excavation, #script, #v2, #output]
source_count: 1
relates_to:
  - page: "[[Run an Analysis]]"
    rel: example_of
  - page: "[[Extract Results]]"
    rel: example_of
  - page: "[[Assign Materials]]"
    rel: example_of
  - page: "[[Output]]"
    rel: example_of
---

# Cantilever Wall FOS (example)

## Summary
The most complete end-to-end V2 script in the shipped sources: build a 2D domain, assign a
Mohr-Coulomb soil and a plate wall, run a factor-of-safety analysis, and extract the
collapse mechanism plus plate sectional forces. This is the reference for "what a real
script looks like."

## Code Example
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py, run_analysis method)
# Set up new project and model
gx = GX()
prj = gx.create_project('Cantilever Wall')
mod = prj.create_model('FOS')

# Materials
c = self.c_spinbox.value()
phi = self.phi_spinbox.value()
gamma = self.gamma_spinbox.value()
mp = self.capacity_spinbox.value()
r = self.roughness_spinbox.value()

Soil = prj.MohrCoulomb(name='Soil', c=c, phi=phi, gamma_dry=gamma, color=rgb(180,180,90))
Wall = prj.GeneralPlate(name='Wall', m_px=mp, color='blue')

# Geometry
H = self.H_spinbox.value()
D = self.D_spinbox.value()
mod.add_line([0,0],[0,H])
mod.add_line([0,H],[2*H,H])
mod.add_line([2*H,H],[2*H,-2*H])
mod.add_line([2*H,-2*H],[-2*H,-2*H])
mod.add_line([-2*H,-2*H],[-2*H,0])
mod.add_line([-2*H,0],[0,0])
mod.add_line([0,0],[0,-D])

# Assign materials
sel = mod.select([0,0], types='face')
mod.set_solid(sel, Soil)

sel = mod.select([0,-D/2], types='edge')
mod.set_plate(sel, Wall, strength_reduction_factor=r)
sel = mod.select([0,H/2], types='edge')
mod.set_plate(sel, Wall, strength_reduction_factor=r)

# Standard Fixities and Zoom All
mod.set_standard_fixities()
mod.zoom_all()

# Analysis settings
mod.analysis_type = 'factor_of_safety'

# Run analysis
prj.run_analysis()

# Extract results
FOS = mod.output.critical_results.factor_of_safety
soil = mod.output.solid
wall = mod.output.plate

# Collapse mechanism
Xs, Ys, Us = [], [], []
for i in range(len(soil)):
    Xs.append(soil[i].topology.X)
    Ys.append(soil[i].topology.Y)
    Us.append(soil[i].results.collapse_mechanism.u_norm.value)

# Plate sectional forces
import numpy as np
Yw = np.array([]); M = np.array([]); V = np.array([]); N = np.array([])
for i in range(len(wall)):
    y = wall[i].topology.Y
    Yw = np.append(Yw, y)
    M = np.append(M, wall[i].results.final_forces.M_y.value)
    V = np.append(V, wall[i].results.final_forces.V_y.value)
    N = np.append(N, wall[i].results.final_forces.N.value)
```

## Related
- [[Assign Materials]] — the `set_solid`/`set_plate` steps.
- [[Run an Analysis]] — `mod.analysis_type` + `prj.run_analysis()`.
- [[Extract Results]] / [[Output]] — the `mod.output.*` reads.

## Notes
- `self.*_spinbox.value()` are Qt widget reads from the host plugin — replace with plain
  numbers when scripting standalone.
- Note the single-model FOS shortcut: `mod.analysis_type` and `mod.output` are used
  without explicitly creating a [[Stage]].
- `prj.GeneralPlate(..., m_px=mp)` passes a plastic-moment kwarg accepted by the server
  even though it's not in the typed signature — see [[Material Models]] Notes.
