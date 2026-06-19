---
title: Remote Sync and Caching
category: concept
source: RemoteObject.py, Materials/RemoteMaterial.py, RemoteStage.py
last_updated: 2026-06-19
tags: [#script, #automation, #v2, #gotcha, #grpc]
source_count: 3
relates_to:
  - page: "[[RemoteMaterial]]"
    rel: supports
  - page: "[[AnalysisProperties]]"
    rel: supports
  - page: "[[Csys]]"
    rel: supports
---

# Remote Sync and Caching

## Summary
Materials ([[RemoteMaterial]]) and stage settings ([[AnalysisProperties]]) are
**remote-synced objects**: reading a property may fetch from the server, and **assigning a
property pushes to the server immediately**. The shared engine (`RemoteCacheMixin` in
`RemoteObject.py`) adds a read cache, dirty tracking, and a `batch()` mode. Understanding
it explains both why scripts "just work" and where the performance traps are.

## Details
`RemoteCacheMixin` provides:
- **Read cache** with a timeout (`_DEFAULT_CACHE_TIMEOUT = 5.0` s). A getter returns the
  cached value unless stale, then calls `_refresh()` (one gRPC pull of all properties).
- **Write-through with dirty tracking.** `_set_property` stores the value, marks it
  dirty, and (unless batching) calls `_sync()` to push it now.
- **`batch()` context manager.** Inside `with obj.batch():`, writes accumulate and a
  single `_sync()` fires on exit — one round trip for many edits.
- `invalidate_cache()`, `is_dirty`, `dirty_properties`, `cache_timeout` setter.

Subclasses implement `_refresh`/`_sync`/`_create` over their specific gRPC calls
(materials → `*_remote_material`; stages → `set/get_analysis_properties`).

## Code Example
```python
# INFERRED — the performance-critical pattern
mat = prj.MohrCoulomb(name='s', phi=30)

# SLOW: 3 separate gRPC round trips
mat.phi = 31
mat.c   = 6
mat.E   = 25000

# FAST: 1 round trip
with mat.batch():
    mat.phi = 31
    mat.c   = 6
    mat.E   = 25000

mat.cache_timeout = 0      # always re-read from server (no stale reads)
mat.invalidate_cache()     # force next read to fetch
```

## Related
- [[RemoteMaterial]] — material instance of this pattern.
- [[AnalysisProperties]] — stage-settings instance of this pattern.
- [[Csys]] — coordinate systems update similarly.

## Notes
- **Gotcha — stale reads:** within the 5 s window a getter may return a cached value even
  if the GUI changed it. Use `cache_timeout=0` or `invalidate_cache()` when you need
  authoritative reads.
- **Gotcha — loop writes:** assigning remote properties inside a loop sends one call per
  assignment. Wrap the loop body in `batch()`.
- Assigning `None` or `...` (Ellipsis) is a no-op — that's how "leave at default" works in
  the factory constructors.
