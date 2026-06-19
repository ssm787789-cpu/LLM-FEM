---
title: Object Hierarchy
category: concept
source: v2.py, GxClient.py
last_updated: 2026-06-19
tags: [#script, #automation, #v2, #stage]
source_count: 2
relates_to:
  - page: "[[GX]]"
    rel: supports
  - page: "[[Project]]"
    rel: supports
  - page: "[[Model]]"
    rel: supports
  - page: "[[Stage]]"
    rel: supports
  - page: "[[V1 vs V2 API]]"
    rel: supports
---

# Object Hierarchy

## Summary
The V2 OptumGX scripting API is organized as a strict containment tree. Knowing which
object owns which capability is the single most useful thing for writing correct
scripts — geometry is *not* on stages, materials are *not* on models, results are *not*
on the project.

## Details

```
GX                         # connection / session         -> api/GX
└── Project                # materials, csys, run_analysis -> api/Project
    └── Model              # GEOMETRY + base settings      -> api/Model
        └── Stage          # analysis phase / settings     -> api/Stage
            └── output     # results (property)            -> api/Output
```

| Capability | Lives on |
|------------|----------|
| Connect / open / save project | [[GX]] |
| Create materials & coordinate systems | [[Project]] |
| Run the analysis | [[Project]] (`run_analysis`) |
| Build/edit geometry | [[Model]] |
| Select shapes, query topology | [[StageBase]] (so both [[Model]] & [[Stage]]) |
| Loads / BCs / structural elements | [[FeatureAPI]] (on [[Model]] & [[Stage]]) |
| Analysis settings (type, mesh, time) | [[AnalysisProperties]] (on [[Stage]] & [[Model]]) |
| Results | [[Output]] (`stage.output`) |

[[Model]] and [[Stage]] share the base class [[StageBase]], which is why selection,
output, solver settings, and analysis properties work on both. A `Model` adds geometry
construction; a `Stage` adds `from_stage` chaining ([[Staged Construction]]).

## Code Example
```python
# INFERRED — the canonical top-to-bottom skeleton
gx   = GX()                               # session
prj  = gx.create_project('demo')          # project
mod  = prj.create_model('M')              # model (geometry)
soil = prj.MohrCoulomb(name='s', phi=30)  # material (on project!)
mod.add_rectangle([0,0],[10,5])           # geometry (on model!)
st   = mod.create_stage('stage 1')        # stage (analysis phase)
st.analysis_type = 'load_multiplier'      # settings (on stage)
prj.run_analysis()                        # run (on project!)
print(st.output.global_results.load_multiplier)  # results (on stage)
```

## Related
- [[GX]], [[Project]], [[Model]], [[Stage]] — the four levels.
- [[StageBase]] — the shared base of Model and Stage.
- [[V1 vs V2 API]] — the legacy flat alternative.
- [[Connect and Set Up a Session]] — the first workflow.

## Notes
- Common beginner error: calling `mod.MohrCoulomb(...)` (materials are on the
  **project**) or `stage.add_line(...)` (geometry is on the **model**).
