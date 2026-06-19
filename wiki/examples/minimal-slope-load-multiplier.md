---
title: Minimal Slope Load Multiplier (example)
category: example
source: v2.py (StageBase.output docstring), inferred
last_updated: 2026-06-19
tags: [#stability, #stage, #script, #v2]
source_count: 1
relates_to:
  - page: "[[Stage]]"
    rel: example_of
  - page: "[[Analysis Types]]"
    rel: example_of
  - page: "[[Build 2D Geometry]]"
    rel: example_of
---

# Minimal Slope Load Multiplier (example)

## Summary
A from-scratch, dependency-free script: a single soil block, a lower-bound
`load_multiplier` stage, and the collapse load read back. Good first script to confirm
your environment and connection.

## Code Example
```python
# INFERRED (structure follows the VERBATIM StageBase.output docstring in v2.py)
from OptumGX import *

gx  = GX()
prj = gx.create_project('slope_demo')
mod = prj.create_model('slope')

# Material
soil = prj.MohrCoulomb(name='Soil', phi=28, c=5, gamma_dry=18, gamma_sat=20)

# Geometry: a 10 x 5 block
mod.add_rectangle([0,0],[10,5])
face = mod.select([5,2.5], types='face')
mod.set_solid(face, soil)

# Surface load that scales with the multiplier
top = mod.select([0,5],[10,5], types='edge', option='green')
mod.set_line_load(top, p=-1, direction='z', option='multiplier')

mod.set_standard_fixities()

# Lower-bound limit analysis stage
stage = mod.create_stage('lower bound')
stage.set_analysis_properties(
    analysis_type='load_multiplier',
    element_type='lower',
    no_of_elements=1000,
)
stage.set_run_flag(RunFlag.run)

prj.run_analysis()
print('collapse load multiplier =',
      stage.output.global_results.load_multiplier)

gx.save_project(r'C:\work\slope_demo.gxx')
```

## Related
- [[Build 2D Geometry]] — the `add_rectangle`/`select` pattern.
- [[Analysis Types]] — `load_multiplier` and lower/upper bounds.
- [[Stage]] — `create_stage`, `set_analysis_properties`, `output`.

## Notes
- Run it twice with `element_type='lower'` then `'upper'` to bracket the true collapse
  load.
- `option='multiplier'` on the load is what makes `load_multiplier` meaningful — a fixed
  load would not scale.
- INFERRED snippet: assembled from documented method signatures, not copied verbatim;
  verify against your OptumGX version.
