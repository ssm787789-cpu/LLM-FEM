---
title: Overview
category: concept
source: synthesis of all ingested sources
last_updated: 2026-06-19
tags: [#script, #automation, #v2, #grpc]
source_count: 12
---

# Overview — Scripting OptumGX

High-level synthesis of the OptumGX Python scripting API. Start here, then drill into
[[index|the index]] and the linked pages.

## What the API is
The OptumGX Python package (`library/OptumGX/`) is a **gRPC client**. The OptumGX
**GUI application is the server** (default `localhost:6000`). Your script connects to a
running GUI and mutates the live model you see on screen — there is no headless solver
in this package. Two API generations share one channel:

- **V2 (preferred):** object hierarchy [[GX]] → [[Project]] → [[Model]] → [[Stage]],
  with auto-syncing materials and stage settings.
- **V1 (legacy):** the flat [[GxClient]] surface. See [[V1 vs V2 API]].

## What you can do
- Build/edit 2D and 3D **geometry** ([[Model]]): lines, polygons, primitives, transforms,
  extrude/revolve/slice ([[Build 2D Geometry]], [[Extrude 2D to 3D]]).
- Create **materials** ([[RemoteMaterialAPI]], [[Material Models]]) — ~27 models — and
  assign them to selected shapes ([[Assign Materials]]).
- Apply **loads, boundary conditions, structural elements, and hydraulic BCs**
  ([[FeatureAPI]], [[Apply Loads and Boundary Conditions]]).
- Configure and run **analyses** of 7 types ([[Analysis Types]], [[Run an Analysis]]),
  including staged construction ([[Staged Construction]]).
- Extract **results** — scalar critical/global values and per-element fields
  ([[Output]], [[Extract Results]]).

## What you cannot (easily) do
- Run without the GUI server (no headless mode here).
- Use a non-default port through `GX()` (it hard-codes 6000 — see [[GX]] Notes).
- Rely on `gx.new.<Material>` from docstrings — that accessor does not exist; use
  `prj.<Material>(...)` ([[RemoteMaterialAPI]] Contradictions).

## Object hierarchy (who owns what)
```
GX        connect / open / save                      -> api/GX
Project   materials, csys, run_analysis              -> api/Project
Model     geometry + base settings + stages          -> api/Model
Stage     analysis_type, settings, from_stage, output-> api/Stage
```
Geometry lives on the **model**, materials on the **project**, settings on the **stage**,
results on `stage.output`. See [[Object Hierarchy]].

## Most important classes for automation (ranked by inbound links)
1. [[Model]] — geometry + the hub most workflows touch.
2. [[Stage]] / [[StageBase]] — analysis phases, selection, output.
3. [[Project]] — materials and the run trigger.
4. [[FeatureAPI]] — loads/BCs/structural elements.
5. [[RemoteMaterialAPI]] / [[RemoteMaterial]] — materials.
6. [[Output]] — results.
7. [[ShapeList]] — the universal selection handle.
8. [[GX]] — the session.

## Key gotchas / version notes
- **Auto-sync:** assigning a material/stage property pushes to the server immediately;
  batch loop edits with `with obj.batch():` ([[Remote Sync and Caching]]).
- **Stale reads:** getters cache ~5 s; use `cache_timeout=0` / `invalidate_cache()` for
  authoritative reads.
- **`output`:** V2 property vs V1 method ([[Output]] Contradictions).
- **Positional shapes:** no stable shape names — re-select by location after edits
  ([[Shapes and Selection]]).
- **Type↔method pairing:** solids→`set_solid`, plates→`set_plate`, etc.

## Suggested first scripts
1. [[Minimal Slope Load Multiplier (example)]] — confirm connection + a limit analysis.
2. [[Cantilever Wall FOS (example)]] — full VERBATIM end-to-end FOS with result plots.
3. [[Read Critical Results (example)]] — scalar result extraction patterns.

## Suggested first 5 queries to test the wiki
1. "How do I connect and create a 2D model?" → [[Connect and Set Up a Session]].
2. "How do I make a Mohr-Coulomb soil and assign it to a face?" → [[Assign Materials]].
3. "How do I run a factor-of-safety analysis and read the FOS?" → [[Run an Analysis]] + [[Output]].
4. "What's the difference between `gx.output()` and `stage.output`?" → [[Output]] Contradictions.
5. "How do I give Young's modulus a depth gradient?" → [[Field Values (ParameterMap, Profile, Gradient)]].

## Suggested first automation script to write
A **parametric FOS sweep**: loop a soil parameter (e.g. `phi` from 20° to 35°), for each
value rebuild the [[Cantilever Wall FOS (example)]] model (or just reassign the material
and re-run), collect `mod.output.critical_results.factor_of_safety`, and write a CSV.
This exercises [[Project]], [[Model]], [[RemoteMaterialAPI]], [[Run an Analysis]], and
[[Output]] end-to-end and is the natural template for design studies.
