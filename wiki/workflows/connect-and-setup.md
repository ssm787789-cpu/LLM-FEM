---
title: Connect and Set Up a Session
category: workflow
source: v2.py, GxClient.py, python311/setup_venv.bat
last_updated: 2026-06-19
tags: [#script, #automation, #grpc, #v2]
source_count: 3
relates_to:
  - page: "[[GX]]"
    rel: example_of
  - page: "[[GxClient]]"
    rel: example_of
  - page: "[[Object Hierarchy]]"
    rel: supports
---

# Connect and Set Up a Session

## Summary
Every script starts by connecting to a **running OptumGX GUI** (the gRPC server) and
obtaining a [[GX]] session. This page covers the environment, the connection, and the
first project/model.

## Details
1. **Interpreter / package path.** The package is `library/OptumGX/`. The shipped
   `python311/setup_venv.bat` and `HelpDev.start()` register the path via an
   `OptumGX.pth` file so `import OptumGX` works. Use the bundled `python311` or any 3.11+
   with `grpcio`, `protobuf`, `numpy`.
2. **Start the server.** Launch the OptumGX GUI (`apps/OptumGX.exe`). It listens on
   `localhost:6000` by default. There is no headless server in this package.
3. **Connect.** `gx = GX()` — opens the channel and wraps the current project context.
4. **Create or open a project**, then a model.

## Steps
```python
# INFERRED — full bootstrap
from OptumGX import *      # GX, GxClient, materials, rgb, enums

gx  = GX()                          # 1) connect (GUI must be running on :6000)
prj = gx.create_project('demo')     # 2) new project (overwrites current!)
mod = prj.create_model('Model 1')   # 3) a model to build geometry in
print('OptumGX', gx.app_version())  # sanity check the link
```

To open an existing file instead:
```python
# INFERRED
import os
prj = gx.open_project(os.path.abspath('slope.gxx'))
mod = prj.get_current_model()
```

## Related
- [[GX]] — the session object and its methods.
- [[GxClient]] — the transport; the only way to set a non-default port.
- [[Object Hierarchy]] — what comes after the session.
- [[Build 2D Geometry]] — the usual next step.

## Notes
- If `GX()` hangs or errors, the GUI is not running, or it is on a different port. `GX()`
  cannot change the port (it hard-codes `GxClient()` on 6000 — see [[GX]] Notes); run the
  GUI on the default port.
- `create_project()` discards the current project silently — `save_project()` first if it
  matters.
