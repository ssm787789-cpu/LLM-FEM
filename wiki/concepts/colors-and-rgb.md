---
title: Colors and rgb
category: concept
source: DataModelV2.py, RemoteObject.py
last_updated: 2026-06-19
tags: [#material, #script, #v2]
source_count: 2
relates_to:
  - page: "[[RemoteMaterialAPI]]"
    rel: supports
  - page: "[[RemoteMaterial]]"
    rel: supports
---

# Colors and rgb

## Summary
Materials (and some features) take a `color=`. The API accepts an `rgb(...)` value, a
named-color string, or an HTML hex string — all converted to the proto `Color` message.
Minor but ubiquitous, since every `prj.<Material>(...)` call usually sets one.

## Details
- `rgb(r, g, b, a=255)` (`DataModelV2.py`) builds a `Color`.
- Named colors: `DataModelV2._stdColors` (OptumGX palette) — `blue, green, orange,
  yellow, gray, red, darkblue, darkgreen, magenta, cyan` (case-insensitive).
- `RemoteObject.to_color` additionally accepts HTML names (`'navy'`, `'teal'`, ...),
  matplotlib `'tab:'`-style names, and hex `'#rrggbb'` / `'#rgb'` / `'#rrggbbaa'`.

## Code Example
```python
# VERBATIM (plugins/Example_Plugin/example_plugin.py)
Soil = prj.MohrCoulomb(name='Soil', c=c, phi=phi, gamma_dry=gamma, color=rgb(180,180,90))
Wall = prj.GeneralPlate(name='Wall', m_px=mp, color='blue')
```
```python
# INFERRED — hex and named forms
m1 = prj.Tresca(name='Clay', cu=40, color='#8844aa')
m2 = prj.LinearElastic(name='Steel', E=2.1e8, nu=0.3, color='gray')
```

## Related
- [[RemoteMaterialAPI]] — constructors take `color=`.
- [[RemoteMaterial]] — `color` is a syncable property you can change later.

## Notes
- An unrecognized color name returns `None` (no color set) rather than raising — check
  spelling if a material shows the default color.
