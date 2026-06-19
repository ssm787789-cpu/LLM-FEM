---
title: GX
category: api
source: v2.py
last_updated: 2026-06-19
tags: [#script, #automation, #v2, #grpc]
source_count: 1
relates_to:
  - page: "[[Project]]"
    rel: required_by
  - page: "[[GxClient]]"
    rel: extends
  - page: "[[Object Hierarchy]]"
    rel: example_of
  - page: "[[Connect and Set Up a Session]]"
    rel: example_of
---

# GX

## Summary
`GX` is the top-level entry point of the **V2** OptumGX scripting API. Constructing it
opens a gRPC connection to a running OptumGX GUI instance and gives you access to
projects. Almost every V2 script starts with `gx = GX()`. It is the modern replacement
for the legacy [[GxClient]].

## Details
Defined in `v2.py` (`class GX`). On construction it internally creates a [[GxClient]]
and reuses its V2 stub:

```python
def __init__(self):
    self._client = GxClient()      # opens grpc channel to localhost:6000
    self._v2 = self._client._v2    # ContractV2 ApiStub
```

### Methods
| Method | Returns | Purpose |
|--------|---------|---------|
| `get_current_project()` | [[Project]] | Wrap the project currently open in the GUI. |
| `create_project(name=None)` | [[Project]] | Create a new project (**overwrites** the current one). |
| `open_project(file_path)` | [[Project]] | Open a `.gxx` project file. |
| `save_project(file_path)` | — | Save current project (overwrites without warning; path is `abspath`-normalized). |
| `write_step(points, faces, fileName)` | — | Export geometry to STEP. |
| `create_report(template_file, data_file, output_file, format_script)` | — | Generate a report. |
| `app_version()` | str | OptumGX version string from the server. |

`GX` has no `port` argument — to use a non-default port construct a [[GxClient]]
directly. See [[Connect and Set Up a Session]].

## Code Example
```python
# INFERRED — minimal V2 session bootstrap
from OptumGX import *          # exposes GX, rgb, material factories, enums

gx = GX()                      # connect to running OptumGX on localhost:6000
prj = gx.create_project('demo')
mod = prj.create_model('Model 1')
# ... build geometry, assign materials, run ...
gx.save_project(r'C:\work\demo.gxx')
```

## Related
- [[Project]] — what every `GX` method returns; the next layer down.
- [[GxClient]] — the transport `GX` is built on; also the V1 API surface.
- [[Object Hierarchy]] — where `GX` sits in `GX → Project → Model → Stage`.
- [[Connect and Set Up a Session]] — workflow for getting a live `gx`.

## Notes
- `GX()` **requires a running OptumGX GUI** acting as the gRPC server; there is no
  headless server start in this package. If connection fails, the GUI is not open or is
  on another port.
- `create_project()` silently discards the current project — save first if needed.
- `from OptumGX import *` (via `__init__.py`) pulls in `GX`, `GxClient`, all material
  classes, `rgb`, `ReduceStrength`, `DeformationAnalysisType`, `ShapeList`.
