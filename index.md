# OptumGX Scripting Wiki — Index

Content catalog of all wiki pages. **Read this first** before answering any query, then
read the linked pages. Organized by category. See [[Overview]] for the synthesis and
[[CLAUDE.md]] for the maintenance schema.

## API Classes
| Page | Summary | Source |
|------|---------|--------|
| [[GX]] | V2 entry point; connects to the GUI, opens/saves projects | v2.py |
| [[Project]] | Holds materials, csys; creates models; runs analyses | v2.py, RemoteMaterialAPI.py |
| [[Model]] | Geometry construction + base settings; owns stages | v2.py |
| [[Stage]] | One analysis phase; settings + `from_stage` + `output` | v2.py, RemoteStage.py |
| [[StageBase]] | Shared base of Model/Stage: select, shapes, solver, output | v2.py |
| [[GxClient]] | gRPC transport + legacy V1 API | GxClient.py, DataModel.py |
| [[ShapeList]] | Shape/ShapeList selection handles | DataModelV2.py |
| [[RemoteMaterialAPI]] | Material factory (constructors + getters) on Project | RemoteMaterialAPI.py |
| [[RemoteMaterial]] | Base class for auto-syncing material objects | RemoteMaterial.py, RemoteObject.py |
| [[RemoteMaterialV1]] | Legacy explicit material create/save | DataModel.py |
| [[MohrCoulomb (material)]] | Workhorse soil model + parameter reference | MohrCoulomb.py |
| [[AnalysisProperties]] | Stage analysis settings as attributes | RemoteStage.py |
| [[FeatureAPI]] | Loads, BCs, structural elements | RemoteFeatures/FeatureAPI.py |
| [[Output]] | Result extraction (critical scalars + per-element) | v2.py, Output.py |
| [[Csys]] | User coordinate systems | v2.py |

## Concepts
| Page | Summary | Source |
|------|---------|--------|
| [[Object Hierarchy]] | GX→Project→Model→Stage and who owns what | v2.py |
| [[V1 vs V2 API]] | Two API generations; prefer V2 | v2.py, GxClient.py |
| [[Material Models]] | Catalog of ~27 models and assignment methods | Materials/*.py |
| [[Analysis Types]] | The 7 `analysis_type` values and their settings | DataModelV2.py |
| [[Field Values (ParameterMap, Profile, Gradient)]] | Spatially varying parameters | Common.py |
| [[Remote Sync and Caching]] | Auto-sync, cache, dirty tracking, batch() | RemoteObject.py |
| [[Shapes and Selection]] | ShapeType + selection model | DataModelV2.py, v2.py |
| [[Staged Construction]] | Phasing with `from_stage` + feature toggles | v2.py |
| [[Drainage and Seepage]] | Material/stage/BC hydraulic knobs | MohrCoulomb.py, RemoteStage.py |
| [[Colors and rgb]] | `rgb()`, named & hex colors | DataModelV2.py |

## Workflows
| Page | Summary | Source |
|------|---------|--------|
| [[Connect and Set Up a Session]] | Server, import, `GX()`, first project | v2.py |
| [[Build 2D Geometry]] | Lines/primitives + selecting faces | v2.py, example_plugin.py |
| [[Assign Materials]] | Create → select → `set_solid`/`set_plate` | RemoteMaterialAPI.py |
| [[Apply Loads and Boundary Conditions]] | Fixities, supports, loads | FeatureAPI.py |
| [[Run an Analysis]] | Settings → run flag → `run_analysis()` | v2.py |
| [[Extract Results]] | Scalars + per-element loops from `output` | Output.py |
| [[Extrude 2D to 3D]] | Section → depths → 3D model | v2.py |

## Examples
| Page | Summary | Source |
|------|---------|--------|
| [[Cantilever Wall FOS (example)]] | Full VERBATIM end-to-end FOS script | example_plugin.py |
| [[Minimal Slope Load Multiplier (example)]] | Dependency-free first script | v2.py + inferred |
| [[Read Critical Results (example)]] | Scalar `CriticalResults` dump | Output.py |
