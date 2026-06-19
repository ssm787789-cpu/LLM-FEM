---
title: Field Values (ParameterMap, Profile, Gradient)
category: concept
source: Common.py, RemoteObject.py, DataModelV2.py
last_updated: 2026-06-19
tags: [#material, #load, #script, #v2]
source_count: 3
relates_to:
  - page: "[[RemoteMaterial]]"
    rel: supports
  - page: "[[MohrCoulomb (material)]]"
    rel: supports
  - page: "[[FeatureAPI]]"
    rel: supports
---

# Field Values (ParameterMap, Profile, Gradient)

## Summary
Many material and load parameters accept not just a constant but a **spatially varying
field**. OptumGX models this with the proto `IField`, and the Python API wraps it in three
helper classes — `Gradient`, `Profile`, `ParameterMap` — that you pass anywhere a
`float | ParameterMap | Profile | Gradient` is accepted (e.g. `E=`, `c=`, load `p=`).

## Details
Defined in `Common.py` (and converted in `RemoteObject.to_ifield`/`from_ifield`).

| Wrapper | Constructor | Meaning |
|---------|-------------|---------|
| constant | plain `float`/`int` | Uniform value. |
| `Gradient(zref, zgrad, value)` | linear in z | `value` at `z=zref`, changing by `zgrad` per unit depth. |
| `Profile(data)` | `[[depth, value], ...]` | Piecewise depth profile. |
| `ParameterMap(data)` | `[[x, y, z, value], ...]` | Arbitrary spatial map of points. |

Reading a property back converts the `IField` to the matching Python wrapper (or a float
if constant), so round-tripping is lossless.

## Code Example
```python
# INFERRED — three ways to specify Young's modulus
mc = prj.MohrCoulomb(name='s', phi=30, c=5)
mc.E = 30000                                  # constant
mc.E = Gradient(zref=0, zgrad=1500, value=20000)   # 20 MPa at surface, +1.5 MPa/m
mc.E = Profile([[0, 20000], [5, 35000], [10, 50000]])   # by depth
mc.Kx = ParameterMap([[0,0,0, 1e-7], [10,0,0, 1e-6]])   # spatially mapped conductivity
```

## Related
- [[RemoteMaterial]] — most field-valued props live on materials.
- [[MohrCoulomb (material)]] — `E`, `c`, `phi`, `Kx`... all accept fields.
- [[FeatureAPI]] — line/surface load `p=` also accepts `ParameterMap`/`Gradient`.

## Notes
- `Gradient` z increases **downward** in the usual geotechnical convention; pick `zgrad`
  sign accordingly.
- These wrappers implement `__eq__`, so cached vs new field values compare correctly and
  the dirty-tracking in [[Remote Sync and Caching]] won't re-send unchanged fields.
