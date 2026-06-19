# OptumGX Wiki — Log

Append-only, chronological. Each entry starts with `## [YYYY-MM-DD] <op> | <title>`
so it is grep-parseable: `grep "^## \[" log.md | tail -5`.

## [2026-06-19] init | Repository created
Created `optumgx-wiki/` structure (CLAUDE.md, index.md, log.md, overview.md, raw/, wiki/{api,concepts,workflows,examples}) and `git init`. Wrote CLAUDE.md schema first.

## [2026-06-19] ingest | library/OptumGX/__init__.py + GxClient.py + Common.py
Identified architecture: `OptumGX` is a gRPC client; GUI is the server on localhost:6000. `GxClient(RemoteObject, RemoteMaterialV1, RemoteMaterialAPI)` carries both V1 (`_api`) and V2 (`_v2`) stubs. Pages: [[GxClient]], [[GX]], [[Colors and rgb]], [[Field Values (ParameterMap, Profile, Gradient)]].

## [2026-06-19] ingest | v2.py
Core V2 object model. Pages: [[GX]], [[Project]], [[Model]], [[Stage]], [[StageBase]], [[ShapeList]], [[Csys]], [[Output]], [[Object Hierarchy]], [[V1 vs V2 API]], [[Staged Construction]], [[Build 2D Geometry]], [[Extrude 2D to 3D]], [[Run an Analysis]].

## [2026-06-19] ingest | RemoteObject.py + RemoteStage.py
Remote-sync engine (cache/dirty/batch) and `AnalysisProperties` stage-settings mixin. Pages: [[Remote Sync and Caching]], [[AnalysisProperties]], [[Analysis Types]].

## [2026-06-19] ingest | DataModelV2.py + Common.py + ContractV2_enum.py
Enums (ShapeType, AnalysisType, drainage/solver/element/load enums), `Shape`/`ShapeList`, `rgb`, field wrappers. Pages: [[Shapes and Selection]], [[ShapeList]], [[Drainage and Seepage]], [[Colors and rgb]], [[Field Values (ParameterMap, Profile, Gradient)]].

## [2026-06-19] ingest | Materials/RemoteMaterialAPI.py + RemoteMaterial.py + MohrCoulomb.py
Material factory, base class, representative model. Pages: [[RemoteMaterialAPI]], [[RemoteMaterial]], [[MohrCoulomb (material)]], [[Material Models]], [[Assign Materials]].

## [2026-06-19] ingest | DataModel.py (V1)
V1 object model + `RemoteMaterialV1`. Pages: [[GxClient]], [[RemoteMaterialV1]], [[V1 vs V2 API]].

## [2026-06-19] ingest | RemoteFeatures/FeatureAPI.py
Loads/BCs/structural features mixin. Pages: [[FeatureAPI]], [[Apply Loads and Boundary Conditions]].

## [2026-06-19] ingest | Output.py + ResultData_pb2.py
Dense-mesh result extraction. Pages: [[Output]], [[Extract Results]], [[Read Critical Results (example)]].

## [2026-06-19] ingest | plugins/Example_Plugin/example_plugin.py
Full VERBATIM end-to-end V2 script. Pages: [[Cantilever Wall FOS (example)]] and verbatim snippets across Model/Project/FeatureAPI/Output workflow pages.

## [2026-06-19] lint | First pass
Flagged 3 contradictions for human review:
1. `gx.new.<Material>` in material docstrings vs no `new` accessor in shipped package (real call: `prj.<Material>(...)`). → [[RemoteMaterialAPI]], [[Material Models]].
2. `output` is a V2 @property (`stage.output`) but a V1 method (`gx.output()`). → [[Output]], [[V1 vs V2 API]].
3. `GX()` hard-codes `GxClient()` on port 6000, ignoring the documented `GxClient(port=...)`. → [[GX]], [[GxClient]].
Gaps noted (no own page yet): PileRow/anchor structural detail, Result Points, Design Approaches, 3D-specific selection, transient seepage time-stepping, GeneralPlate parameter reference, mesh adaptivity tuning. Orphan check: none (all pages reachable from [[index|index.md]] / [[Overview]]).
