# ChipletTherm

**Fast, FEM-grade static & transient thermal analysis for 2.5D / 3D chiplet designs.**

ChipletTherm computes full-chip, full-stack temperature fields for heterogeneous-integration
designs — every die, bonding layer, and interposer resolved in 3D — at a fraction of the cost of a
full 3D finite-element (FEM) solve. It is powered by a fast **spectral** engine: the heat equation is decoupled laterally with a spectral transform
and resolved through the stack with a layer-aware thickness model, so steady-state maps come back
in milliseconds and transients in well under a second — validated throughout against a consistent-mass
3D-FEM reference.

This repository hosts the **ChipletTherm** promotion site live at **<https://sheldonucr.github.io/chipletTherm_io/>**.

> **Naming.** The tool is **ChipletTherm**, and it ships in two solver modes:
> **ChipletTherm (TASTA)** — the thickness-resolved, most-accurate solver — and **ChipletTherm-2D
> (TASTA-2D)** — the fast, layer-averaged solver. *TASTA* / *TASTA-2D* are the names used in our paper
> and figures (*Fast2D-FD* is an earlier internal name for ChipletTherm-2D). This README and the site
> use the **ChipletTherm** names, with the paper names in parentheses where figures reference them.

---

## Why ChipletTherm

3D/2.5D integration makes thermal behavior a first-order design constraint — and with accelerators now
pushing 700–1200 W, the tools accurate enough to trust are too slow to keep in the design loop:

- **Hotspots hide in the stack.** Vertical stacking raises thermal resistance and traps heat between
  layers; logic chiplets next to stacked HBM create localized hotspots that decide whether a design ships.
- **3D FEM is too slow to iterate.** A single full FEM solve discretizes the whole volume into millions
  of unknowns — fine for one sign-off, impossible for the thousands of evaluations a design-space search
  (or a runtime control loop) demands.
- **Thermal must be in the loop.** Floorplanning, power delivery, packaging co-design, and runtime
  management all need temperature feedback *per iteration*.

ChipletTherm closes that gap: **best-in-class accuracy at hundreds-to-thousands× the speed.**

---

## Headline results

### Static (steady-state) — 18 cases vs. 3D-FEM reference

ChipletTherm (TASTA) is more accurate than every evaluated method while running orders of magnitude faster:

| Method | Avg RMSE | Avg runtime | Notes |
| --- | --- | --- | --- |
| FEM-3D | — (reference) | 12.7 s | ground truth |
| SOV (baseline) | 0.225 K | 0.047 s | separation of variables |
| GIT (baseline) | 0.219 K | 0.348 s | generalized integral transform |
| **ChipletTherm-2D** (TASTA-2D) | 0.444 K | **0.016 s** | fastest of all methods |
| **ChipletTherm** (TASTA) | **0.214 K** | 0.020 s | most accurate of all methods |

- **ChipletTherm (TASTA) runs 2.42× / 21.4× / 637× faster** than SOV / GIT / FEM-3D (on average), while being more
  accurate than both semi-analytical baselines.
- **Up to 1410× faster** than FEM-3D on the 11-layer 3D-IC stack.
- The thickness-resolved step cuts ChipletTherm-2D's error by ~52% (RMSE 0.444 K → 0.214 K) at essentially
  the same runtime.
- **Real-time capable:** 0.020 s/evaluation → up to ~50 Hz, vs. 1–10 Hz control loops.

**ChipletTherm speedup over FEM-3D, by design:**

| Design | Layers | ChipletTherm RMSE | FEM-3D time | ChipletTherm time | Speedup |
| --- | --- | --- | --- | --- | --- |
| 11-layer 3D IC | 11 | 0.145 K | 33.6 s | 0.027 s | **1410×** |
| 2.5D chiplet package | 9 | 0.099 K | 18.7 s | 0.024 s | 872× |
| HBM3 stack | 6 | 0.041 K | 8.93 s | 0.019 s | 586× |
| 5 nm CPU package | 7 | 0.909 K | 9.52 s | 0.020 s | 563× |
| GaAs RF PA | 5 | 0.009 K | 4.25 s | 0.018 s | 304× |
| 3-layer stack | 3 | 0.081 K | 1.01 s | 0.014 s | 89× |

Validated under 3 power inputs including **measured commercial devices** (Qualcomm Snapdragon 680,
Google Coral M.2 TPU).

### Transient — 12 cases vs. 3D-FEM reference

ChipletTherm-2D (TASTA-2D) vs. a consistent-mass FEM-3D reference, three design families driven by real CPU, GPU,
and TPU power traces, 100 time steps each:

| Metric | Result |
| --- | --- |
| Mean solve-time speedup | **1036×** (range 252× – 1875×) |
| Peak speedup | **1875×** (Chiplet 2.5D, Edge-TPU trace) |
| ChipletTherm-2D solve time | **0.12 – 0.45 s** |
| FEM-3D reference solve time | 30 – 733 s |
| Mean RMSE vs FEM-3D | **1.22 K** (best 0.57 K, worst 2.09 K) |
| Mean temperature error | **0.18%** (range 0.108% – 0.287%) |

| Design family | RMSE | Mean error |
| --- | --- | --- |
| Chiplet 2.5D | 0.72 K | 0.12% |
| CPU 5nm | 1.42 K | 0.24% |
| 3-layer 3D | 1.52 K | 0.23% |
| **All 12 cases** | **1.22 K** | **0.18%** |

---

## Features

- **Resolved 3D static heat maps** — full-chip, full-stack steady-state temperature fields for 2.5D and
  3D assemblies, resolved through-thickness, from a single power + floorplan description.
- **Transient response** — drive the model with arbitrary power waveforms and recover the complete
  temperature history at every node (throttling, workload bursts, thermal transients).
- **Real-time fast** — ~0.02 s steady-state solves and sub-second transients; up to 1410× (static) /
  1875× (transient) faster than 3D FEM; ~50 Hz runtime-management capable.
- **Best-in-class fidelity** — 0.21 K mean RMSE vs a consistent-mass 3D-FEM reference, beating the SOV
  and GIT baselines, across 30 cases including measured Snapdragon and Coral-TPU power maps.
- **Real chiplet stacks** — validated to 11 layers; 2.5D interposers, 3D logic stacks, HBM, CPU and RF
  packages, with per-layer **anisotropic** materials, **interface thermal resistance**, volumetric heat
  capacity, and **Robin** boundary conditions.
- **Batched & scriptable** — independent lateral modes batch naturally across multi-core CPUs and GPUs.
- **Agentic EDA flow ready** — a first-class CLI and structured data interface let autonomous EDA
  agents invoke ChipletTherm, consume machine-readable temperature maps and margins, and feed thermal
  results back into floorplanning, stack planning, power budgeting, and optimization loops.

---

## How it works

ChipletTherm is built on a **spectral fast-analysis** technique. Instead of solving one enormous 3D
system the way full FEM does, it analyzes the chip in a form where the physics nearly separates:

1. **Spectral decomposition** — the in-plane temperature field is broken into a set of simple,
   wave-like spatial modes via a spectral transform. This turns one large, tightly-coupled 3D
   problem into many small, completely independent ones.
2. **Layer-aware vertical model** — each mode is resolved down through the physical stack, capturing
   every die, bonding layer, and interposer with their real per-layer materials, directional
   conductivity, and interface resistances.
3. **Massively parallel solve** — because the modes are independent, they are all solved at once, in a
   single direct pass with no iteration — fast, and a natural fit for multi-core CPU and GPU hardware.
4. **Reassembly** — the solved modes recombine into the full temperature map: the steady-state field
   directly, or advanced step-by-step through time for transient analysis.

Two solver modes span the speed–accuracy tradeoff: **ChipletTherm-2D** (TASTA-2D — layer-averaged, the
fastest, lowest-cost estimate) and **ChipletTherm** (TASTA — thickness-resolved, the most accurate,
recovering the full 3D field). What makes the approach novel is the spectral decoupling: it replaces the expensive 3D solve of
full FEM — and the costlier thickness treatments of prior spectral methods (SOV, GIT) — with hundreds of
tiny, independent solves that run in parallel, delivering lower cost and higher accuracy at the same time.

A full 3D finite-element solver ships alongside as the ground-truth reference, and every ChipletTherm
result is validated against it.

---

## Agentic flow integration

ChipletTherm is built to run inside autonomous, agent-driven EDA flows:

- **Fully agentic-flow aware** — designed to be driven by autonomous EDA agents, and to work with *any*
  agentic flow.
- **First-class CLI** — every analysis is scriptable from the command line; no GUI in the loop.
- **Structured data interface** — machine-readable inputs, temperature fields, hotspot locations, and
  thermal margins for closed-loop automation.
- **Thermal-aware design iteration** — agents can sweep floorplans, stack-ups, power maps, and cooling
  assumptions, then use ChipletTherm results to steer the next candidate.

```text
Agentic EDA flow
   │  invokes ChipletTherm  (CLI + structured data interface)
   ▼
ChipletTherm thermal analysis — static and transient solvers, headless
   │  structured, machine-readable results
   ▼
Fed back to the agent → floorplan, stack, or power-budget iteration repeats
```

Thermal analysis becomes a callable step inside the agentic flow — not a hand-run GUI task.

---

## Figures / assets

The site embeds real **TASTA-vs-FEM-3D** figures from the paper. They live in `assets/figs/` and are
referenced by relative path; if a file is missing, the page falls back to a tasteful placeholder
instead of a broken image, so it always renders. Copy the paper's PNGs into `assets/figs/` before
publishing:

| File | Where it appears on the page |
| --- | --- |
| `figure_array_tpu_0_pd_new.png` | Static results — all six designs, ChipletTherm-2D vs FEM-3D + error maps |
| `fast2d_fd_results_3d_temperature_qual_hbm3.png` | Static results — HBM3 3D field (ChipletTherm-2D) |
| `fem3d_results_3d_temperature_qual_hbm3.png` | Static results — HBM3 3D field (FEM-3D reference) |

Commit the `assets/` folder so the figures are served on GitHub Pages too. (Optionally compress the
PNGs first — the comparison array is fairly large.)

### Transient videos

The transient section embeds two rows of result **videos** (`.mp4`), one per run — **Chiplet 2.5D ·
Intel Core i5 (FLAC)** and **Chiplet 2.5D · Intel Core i7 (GIMP)** — in `assets/figs/videos/`. Each run
contributes three clips (3D temperature field, 3D power density, 2D power density), copied in with a
unique, combination-prefixed name so runs never collide:

| Clip | File name in `assets/figs/videos/` |
| --- | --- |
| 3D temperature field | `chiplet_i5_flac_3D_volume3d.mp4`, `chiplet_i7_gimp_3D_volume3d.mp4` |
| 3D power density | `chiplet_i5_flac_power_density_3d.mp4`, `chiplet_i7_gimp_power_density_3d.mp4` |
| 2D power density | `chiplet_i5_flac_power_density.mp4`, `chiplet_i7_gimp_power_density.mp4` |

The `<video src>` paths in `index.html` reference exactly these names. To add another run, copy its
three clips with a new `<design>_<cpu>_<workload>` prefix and add a matching row. The videos autoplay
muted and loop; if any are missing, that row shows labeled placeholders instead.

## The website

`index.html` is a single, self-contained marketing page (no build step, no web fonts; the only
external files are the figures in `assets/figs/`). To preview locally, open the file directly in a browser,
or serve the folder:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

To publish with **GitHub Pages**: push this repo (including `assets/`), then enable Pages
(Settings → Pages → Deploy from branch → `main` / root). The site is served from `index.html`.

---

## Access

ChipletTherm is in active development. To request access or a walkthrough on your own 2.5D/3D chiplet
designs, contact **stan@ece.ucr.edu**.

---

## Citation

If you use ChipletTherm / TASTA in academic work, please cite:

> J. Lu, S. Racha, and S. X.-D. Tan, "TASTA: Fast Spectral Thermal Solvers for Multilayer 3D IC
> Packages via DCT Decomposition and Layer-Aware Thickness Modeling," University of California,
> Riverside.

```bibtex
@article{lu_tasta_chiplettherm,
  title   = {TASTA: Fast Spectral Thermal Solvers for Multilayer 3D IC Packages
             via DCT Decomposition and Layer-Aware Thickness Modeling},
  author  = {Lu, Jincong and Racha, Shanmukh and Tan, Sheldon X.-D.},
  school  = {University of California, Riverside},
  note    = {NSF CCF-2007135, CCF-2113928}
}
```

---

## Contact

**stan@ece.ucr.edu** · Developed in the Department of Electrical and Computer Engineering,
University of California, Riverside.
