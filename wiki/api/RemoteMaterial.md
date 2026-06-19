---
title: RemoteMaterial
category: api
source: Materials/RemoteMaterial.py, RemoteObject.py
last_updated: 2026-06-19
tags: [#material, #script, #v2, #automation, #gotcha]
source_count: 2
relates_to:
  - page: "[[RemoteMaterialAPI]]"
    rel: required_by
  - page: "[[Remote Sync and Caching]]"
    rel: example_of
  - page: "[[MohrCoulomb (material)]]"
    rel: extends
---

# RemoteMaterial

## Summary
`RemoteMaterial` is the base class for every material object returned by
[[RemoteMaterialAPI]]. It implements the **remote-sync property pattern**: read a
property and it fetches from the server (with a short cache); assign a property and it
**pushes to the server immediately**. This is why `material.phi = 35` "just works"
without an explicit save.

## Details
Defined in `Materials/RemoteMaterial.py`; sync machinery from `RemoteObject.py`
([[Remote Sync and Caching]]).

### Lifecycle methods (internal, underscore-prefixed)
| Method | Effect |
|--------|--------|
| `_create()` | Create on server, receive a UUID; clears cache so next read is fresh. |
| `_sync()` | Push dirty properties via `update_remote_material`. |
| `_refresh()` | Pull all properties via `get_remote_material`. |
| `_delete()` | Remove from server. |
| `_discard_changes()` | Drop local edits, re-fetch. |

You rarely call these directly — assignment triggers `_sync`, construction triggers
`_create`.

### Public surface
- `uuid` (read-only), `category` (`MaterialCategory`), `client`.
- `batch()` context manager — defer sync, push all edits in one call.
- Each subclass adds typed properties via `prop('name', type)` descriptors, mapped to
  nested proto messages through `_PROPERTY_PATHS` (e.g. `'phi' -> ('mohr_coulomb','phi')`).

### Field-valued properties
Strength/stiffness params accept a plain `float` **or** a spatially varying
[[Field Values (ParameterMap, Profile, Gradient)|field]]: `ParameterMap`, `Profile`,
or `Gradient`.

## Code Example
```python
# VERBATIM (Materials/MohrCoulomb.py docstring) — auto-sync + batch
material = gx.new.MohrCoulomb(name="Soil", phi=30.0)   # NOTE: see RemoteMaterialAPI
                                                       # Contradictions re: gx.new
material.phi = 35.0     # synced to server immediately
material.c   = 10.0     # synced again

with material.batch():  # one round trip for all three
    material.phi = 35.0
    material.c   = 15.0
    material.E   = 50000.0
```

## Related
- [[RemoteMaterialAPI]] — constructs and returns these objects.
- [[Remote Sync and Caching]] — the cache/dirty/batch mechanics shared with stages.
- [[MohrCoulomb (material)]] — a concrete subclass and its parameters.
- [[Field Values (ParameterMap, Profile, Gradient)]] — non-constant property values.

## Notes
- **Read cache:** getters cache for ~5 s (`_DEFAULT_CACHE_TIMEOUT`); set
  `material.cache_timeout = 0` to always re-fetch, or call `invalidate_cache()`.
- Assigning inside a tight loop sends one gRPC call per assignment — prefer `batch()`.
- `color` is stored as `object`; pass `rgb(...)` or a color name (see [[Colors and rgb]]).
