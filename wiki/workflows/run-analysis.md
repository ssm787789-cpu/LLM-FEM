---
title: Run an Analysis
category: workflow
source: v2.py, plugins/Example_Plugin/example_plugin.py
last_updated: 2026-06-19
tags: [#solver, #stage, #script, #v2, #automation]
source_count: 2
relates_to:
  - page: "[[Project]]"
    rel: example_of
  - page: "[[Stage]]"
    rel: supports
  - page: "[[Analysis Types]]"
    rel: supports
  - page: "[[Extract Results]]"
    rel: required_by
---

# Run an Analysis

## Summary
Configure the stage(s), flag what to run, then call `prj.run_analysis()`. Results become
available on each [[Stage]]'s `output` property afterward.

## Details
1. Set `analysis_type` and settings on the model/stage ([[AnalysisProperties]],
   [[Analysis Types]]).
2. (Multi-stage) ensure each stage to solve has `set_run_flag(RunFlag.run)`; chain with
   `from_stage` ([[Staged Construction]]).
3. (Optional) `set_solver_settings(solver_type, solver_method)` —
   `solver_method` ∈ `Mosek | Clarabel | QDLDL | FAER | MKL | Panua`.
4. `prj.run_analysis()` (blocks until done).
5. Read results from `stage.output` ([[Extract Results]]).

## Steps
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py) — single-model FOS run
mod.analysis_type = 'factor_of_safety'
prj.run_analysis()
FOS = mod.output.critical_results.factor_of_safety
```
```python
# INFERRED — explicit settings + run flag + solver choice
s = mod.create_stage('limit')
s.set_analysis_properties(analysis_type='load_multiplier',
                          element_type='lower', no_of_elements=2000)
s.set_run_flag(RunFlag.run)
s.set_solver_settings(solver_type=SolverType.standard, solver_method='Mosek')
prj.run_analysis()
print(s.output.global_results.load_multiplier)
```

## Related
- [[Project]] — `run_analysis()` lives here.
- [[Analysis Types]] — what to compute.
- [[AnalysisProperties]] — settings driving the run.
- [[Extract Results]] — reading `output` afterward.

## Notes
- `run_analysis()` solves all run-flagged stages of the current project. Setting
  `RunFlag.none` excludes a stage.
- On a single-model FOS (as in the verbatim example) you can set `mod.analysis_type`
  directly and read `mod.output` without explicitly creating a stage.
- Adaptivity (`mesh_adaptivity=True`, `adaptivity_iterations`) refines the mesh during the
  run — improves limit-analysis bounds at extra cost.
