# ThermStack (ChipletTherm)

**Fast, FEM-grade static & transient thermal analysis for 2.5D / 3D chiplet designs.**

ThermStack (formerly ChipletTherm) computes full-chip, full-stack temperature fields for
heterogeneous-integration designs — every die, bond line, TSV field, interposer and lid resolved in
3D on the design's own **IEEE 3Dblox** geometry — at a fraction of the cost of a full 3D
finite-element or finite-volume solve. It is powered by a fast proprietary engine whose cost is
decoupled from how finely the geometry is rasterized, so a package can be resolved at 128x128
laterally while the solve itself stays small.

This repository hosts the **ThermStack** promotion site live at
**<https://sheldonucr.github.io/chipletTherm_io/>**.

> **Naming.** The tool is **ThermStack** (previously **ChipletTherm**). A 3D finite-volume solver
> (**FDM-3D**) and a consistent-mass 3D finite-element solver (**FEM-3D**) ship alongside as
> ground-truth references, and every result below is measured against both on identical geometry.

---

## Why ThermStack

3D/2.5D integration makes thermal behavior a first-order design constraint — and with accelerators
now pushing 700–1200 W, the tools accurate enough to trust are too slow to keep in the design loop:

- **Hotspots hide in the stack.** Vertical stacking raises thermal resistance and traps heat between
  layers; logic chiplets next to stacked HBM create localized hotspots that decide whether a design
  ships.
- **Volumetric solvers are too slow to iterate.** FEM and FVM must mesh the whole volume, so their
  cost is tied to lateral resolution — and the packages that most need resolving are exactly the ones
  that make them unaffordable.
- **Thermal must be in the loop.** Floorplanning, power delivery, packaging co-design, and runtime
  management all need temperature feedback *per iteration*.

ThermStack closes that gap: **reference-grade accuracy at a fraction of the cost.**

---

## Headline results

### Static (steady-state) — 8 native 3Dblox packages

Both references were run on the identical geometry at three lateral resolutions
(31x31-41x39 through 63x63-73x69); agreement is measured against FEM-3D at 63x63-73x69.

| Method | Lateral grid | Avg RMSE vs FEM-3D | Avg runtime |
| --- | --- | --- | --- |
| FEM-3D <sub>reference</sub> | 63x63-73x69 | — | 153.1 s |
| FDM-3D <sub>finite volume</sub> | 63x63-73x69 | 0.492 K | 71.9 s |
| **ThermStack** | 128x128 | **0.509 K** | **1.93 s** |

- On the 6 conventional packages ThermStack is within **0.004 K** of FEM-3D, where
  the finite-volume solver on the same geometry sits 0.569 K away. The 0.509 K
  average above is carried entirely by the two dense 7 nm packages, where the references
  are themselves still moving with grid.
- Average speedup **68×** over FEM-3D and **31×** over FDM-3D at 63x63-73x69.
- RMSE against the finite-volume reference: 0.967 K — the FVM/FEM scheme gap itself.

**Per design:**

| Design | z bands | ThermStack RMSE | FEM-3D time | FDM-3D time | ThermStack time | Speedup |
| --- | --- | --- | --- | --- | --- | --- |
| 11-layer 3D IC | 55 | 0.003 K | 545.0 s | 248.1 s | 3.00 s | **182×** |
| 2.5D chiplet package | 45 | 0.002 K | 227.1 s | 144.2 s | 2.45 s | **93×** |
| 5 nm CPU package | 35 | 0.004 K | 146.5 s | 68.8 s | 1.81 s | **81×** |
| HBM3 stack | 30 | 0.001 K | 117.6 s | 50.7 s | 1.62 s | **73×** |
| GaAs RF PA | 25 | 0.000 K | 57.7 s | 23.0 s | 1.26 s | **46×** |
| 7 nm HBM+CPU | 125 | 3.158 K | 89.6 s | 26.8 s | 2.34 s | **38×** |
| 3-layer stack | 15 | 0.002 K | 14.1 s | 6.1 s | 0.81 s | **17×** |
| 7 nm HBM+CPU, stepped | 115 | 0.904 K | 26.8 s | 7.6 s | 2.14 s | **12×** |

**What the references cost as the grid refines** (7 nm HBM+CPU, stepped):

| Grid | Cells | FDM-3D peak | FDM-3D time | FEM-3D peak | FEM-3D time |
| --- | --- | --- | --- | --- | --- |
| 39x37x115 | 165,945 | 347.2 K | 0.7 s | 349.0 K | 3.5 s |
| 55x53x115 | 335,225 | 347.2 K | 3.3 s | 348.1 K | 10.8 s |
| 71x67x115 | 547,055 | 347.2 K | 7.6 s | 347.5 K | 26.8 s |
| **128x128 (ThermStack)** | 1,884,160 | **347.6 K** | **2.14 s** | — | — |

### Transient — 8 packages x 100 time steps

Every package driven by the same measured Intel Core i5 (FLAC) power trace,
rasterized onto its own 3Dblox geometry, started from the matching steady state.

| Design | FEM-3D | FDM-3D | ThermStack | Speedup vs FEM-3D | RMSE vs FEM-3D | RMSE vs FDM-3D |
| --- | --- | --- | --- | --- | --- | --- |
| 7 nm HBM+CPU | 303.1 s | 28.9 s | 12.7 s | **24×** | 5.713 K | 5.976 K |
| 11-layer 3D IC | 346.1 s | 40.8 s | 15.5 s | **22×** | 0.233 K | 0.476 K |
| 2.5D chiplet package | 292.5 s | 35.8 s | 14.6 s | **20×** | 0.162 K | 0.143 K |
| GaAs RF PA | 172.2 s | 17.3 s | 9.6 s | **18×** | 0.019 K | 0.060 K |
| HBM3 stack | 206.9 s | 24.3 s | 11.7 s | **18×** | 0.071 K | 1.478 K |
| 5 nm CPU package | 232.4 s | 25.3 s | 14.0 s | **17×** | 0.298 K | 0.785 K |
| 3-layer stack | 98.3 s | 9.7 s | 6.2 s | **16×** | 0.129 K | 0.626 K |
| 7 nm HBM+CPU, stepped | 182.7 s | 17.1 s | 11.8 s | **15×** | 4.386 K | 2.953 K |

- Mean speedup **19×** vs FEM-3D, **2.0×** vs FDM-3D; best **24×**.
- Mean volume RMSE over all steps: **1.376 K** vs FEM-3D, 1.562 K vs FDM-3D.

---

## Features

- **Resolved 3D static heat maps** — full-chip, full-stack steady-state temperature fields for 2.5D
  and 3D assemblies, resolved through-thickness, from a single power + floorplan description.
- **Transient response** — drive the model with arbitrary power waveforms and recover the complete
  temperature history at every node (throttling, workload bursts, thermal transients).
- **Geometry-resolved** — the design's real structure, including **non-coplanar stepped tops**, with
  per-layer **anisotropic** materials, in-plane material variation, **interface thermal resistance**,
  volumetric heat capacity, and **Robin** boundary conditions.
- **Predictable cost** — one structured direct solve, no iteration and no convergence tuning; the
  same design always costs the same.
- **IEEE 3Dblox input** — accepts the IEEE-standard modular description language for physical
  stacking, dimensions, and logical connectivity in 2.5D and 3D-IC designs.
- **Bundled references** — FDM-3D (finite volume) and FEM-3D ship with the tool, so any result can be
  re-checked against ground truth on the same geometry.
- **Agentic EDA flow ready** — a first-class CLI and structured data interface let autonomous EDA
  agents invoke ThermStack, consume machine-readable temperature maps and margins, and feed thermal
  results back into floorplanning, stack planning, power budgeting, and optimization loops.

---

## Figures / assets

The site embeds real **ThermStack-vs-reference** validation figures from `assets/figs/`; if a file is
missing the page falls back to a placeholder instead of a broken image.

| File | Where it appears on the page |
| --- | --- |
| `design_hbm_cpu_7nm_stepped_structure_3d.png` | Static results — the 3Dblox structure of the stepped 7 nm package |
| `thermstack_3d_hbm_cpu_7nm_stepped.png` | Static results — ThermStack 3D field |
| `fem3d_3d_hbm_cpu_7nm_stepped.png` | Static results — FEM-3D reference, same case |
| `grid_refinement_7nm_stepped.png` | Static results — reference cost vs lateral resolution |
| `transient_peak_traces.png` | Transient — measured peak-temperature traces |

### Transient videos

The transient section embeds three clips in `assets/figs/videos/`, all of the **7 nm stepped
HBM+CPU** package under a measured Intel Core i5 (FLAC) trace at a 150 W package budget, 100 steps,
peak 398.5 K:

| Clip | File name in `assets/figs/videos/` |
| --- | --- |
| 3D temperature field (ThermStack output) | `stepped_7nm_thermstack_3d_temperature.mp4` |
| 3D power density (input) | `stepped_7nm_power_density_3d.mp4` |
| 2D power density (input) | `stepped_7nm_power_density_2d.mp4` |

All three are rendered on the design's own 3Dblox structure by `3dblox/plot_3dblox_therm.py`
(the 2D map is the through-stack areal power density). Regeneration scripts and the raw
per-design tables live in `thermstack_io_results/`.

## The website

`index.html` is a single, self-contained page (no build step, no web fonts; the only external files
are the figures in `assets/figs/`). To preview locally:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

To publish with **GitHub Pages**: push this repo (including `assets/`), then enable Pages
(Settings → Pages → Deploy from branch → `main` / root).

---

## Access

ThermStack is in active development. To request a demo or a walkthrough on your own 2.5D/3D chiplet
designs, contact **noveetyai@noveetymanagement.com**.

---

## Contact

**noveetyai@noveetymanagement.com** · © 2026 NoveetyAI, Inc. All rights reserved.

<sub>Benchmarks: static — 8 native IEEE 3Dblox packages, three lateral grids, 0.509 K avg
RMSE vs FEM-3D / 68× avg speedup; transient — 8 packages x 100 steps, 1.376 K RMSE / 19× mean speedup.</sub>
