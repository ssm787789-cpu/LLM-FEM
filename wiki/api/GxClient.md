---
title: GxClient
category: api
source: GxClient.py, DataModel.py
last_updated: 2026-06-19
tags: [#grpc, #v1, #script, #material, #load]
source_count: 2
relates_to:
  - page: "[[GX]]"
    rel: supersedes
  - page: "[[RemoteMaterialV1]]"
    rel: extends
  - page: "[[Connect and Set Up a Session]]"
    rel: example_of
---

# GxClient

## Summary
`GxClient` is the **gRPC transport** and the **legacy V1 API** in one class. It opens the
channel to `localhost:<port>` (default `6000`) and exposes a flat set of V1 methods
(geometry, loads, BCs, materials). In modern scripts it mostly matters as the thing
[[GX]] wraps — but it is the only entry point that takes a custom **port**.

## Details
Defined in `GxClient.py` as
`class GxClient(RemoteObject, RemoteMaterialV1, RemoteMaterialAPI)`.

```python
def __init__(self, port: int = 6000) -> None:
    channel = grpc.insecure_channel(f'localhost:{port}', options=[
        ('grpc.max_send_message_length', 1024**3),
        ('grpc.max_receive_message_length', 1024**3)])
    self._api = ApiStub(channel)        # V1 stub  (Contract_pb2)
    self._v2  = v2svc.ApiStub(channel)  # V2 stub  (ContractV2_pb2)
```

So **one `GxClient` carries both V1 (`._api`) and V2 (`._v2`) stubs** over the same
1 GiB-limit channel. [[GX]] reuses `self._v2`.

### Representative V1 methods (flat, no Project/Model objects)
- Geometry: `add_line`, `add_polygon`, `add_polygons`, `extrude`, `revolve`,
  `create_new_model`, `select`, `pick`, `del_shapes`.
- Materials: inherited `create_material(...)` from [[RemoteMaterialV1]].
- Assignment: `set_solid`, `set_plate`, `set_nailrow`, `set_shear_joint`, `add_connector`.
- Loads/BCs: `set_point_load`, `set_line_load`, `set_surface_load`, `set_body_load`,
  `set_support`, `set_platebc`, `set_standard_fixities`.
- Analysis: `set_analysis_properties(...)`, `set_solver_settings(...)`, `run_analysis()`.
- Results: `results` / `result` (property) → `Result`; `output()` → **method** (V1).
- Files: `open_file`, `save_file`.

### V1 enums (DataModel.py)
`RunFlag(none=0, run=1)`, `AnalysisType(mesh, initial_stresses, load_multiplier,
deformation, load_deformation, factor_of_safety, seepage)`,
`ShapeType(vertex, edge, face, volume)`, `ModelType`.

## Code Example
```python
# INFERRED — connect on a non-default port, then hand to V2
client = GxClient(port=6001)
gx = GX()                 # NOTE: GX() always builds its own GxClient on 6000
# For a custom port with the V2 object model you must drive client._v2 yourself,
# or run the GUI on the default 6000 and just use GX().
```

## Related
- [[GX]] — the V2 entry point that wraps a `GxClient`.
- [[RemoteMaterialV1]] — V1 material creation mixed in here.
- [[V1 vs V2 API]] — when to use which.
- [[Output]] — note the V1 `output()`-method vs V2 `output`-property difference.

## Notes
- **Gotcha:** `GX().__init__` hard-codes `GxClient()` on port 6000 with no override, so
  the documented `GxClient(port=...)` is only honored if you use the V1 surface (or the
  raw `_v2` stub) directly. Flagged on [[GX]] and [[V1 vs V2 API]].
- Prefer [[GX]]/[[Project]]/[[Model]]/[[Stage]] (V2) for new work; V1 remains for
  back-compat and older example scripts.
