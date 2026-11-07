# ThermStack website

Customer-facing product overview and feature guide for ThermStack thermal analysis.
Static HTML, CSS and JavaScript; no build step, external fonts or runtime dependencies.

## Information architecture

The site has three areas: **Features & capabilities**, **White papers** and **Contact**.

- `index.html`: the short main page — cover, capability highlights, workflow, the white-paper
  index (`#whitepapers`) and the contact band. It holds no tables and one technical figure:
  the `#resolution` band shows `assets/figs/apr/apr_20um_overview.png`, which WP-104 also uses
  (the one deliberate exception to "each figure in one place").
- `whitepapers/wp-1xx-*.html`: the white-paper series. All technical details, benchmarks,
  figures, tables and scope notes live here, each figure/table in exactly one paper:
  - `wp-101-numerical-engine.html` — numerical workflow, model inputs/3Dblox, transient analysis.
  - `wp-102-amd-resolution-study.html` — AMD-inspired study: workloads, 250/50 µm comparison,
    interactive result views.
  - `wp-103-gnn-surrogate.html` — ThermStack-GNN benchmark and usage scope.
  - `wp-104-resolution-on-demand.html` — ThermStack APR, resolution on demand: MI350-inspired
    package with synthetic 10 µm GPU power maps, all XCDs at 20 µm, wall time and memory only (no coarse-grid results).
    Results only; the APR method is not described.
  - `wp-105-chip-package-board.html` — chip, package and board in one model: MI350-class OAM
    module from a 3Dblox chip stack and IPC-2581 package and board files, design specs, APR 50 µm
    result with wall time and memory. Input formats and results only; no method, no tool comparisons.
  - `wp-106-layout-level-resolution.html` — layout-level resolution: a placed-and-routed RISC
    processor core resolved to 50 nm (APR), accuracy vs a full 1 µm single grid, wall time and memory.
    Key numbers only; no PDK, design files, tools or method are named.
  New papers follow the same template: `paper-meta` header, `paper-abstract`, numbered
  `feature-section` blocks, a closing scope section and the series `paper-nav`.
- `features.html`: redirect stub only — its content moved into the white papers; old anchors
  (`#numerical`, `#amd-study`, `#gnn`, `#apr`, `#resolution`, `#oam`, …) forward to the right paper.
- `assets/site.css` and `assets/site.js`: shared responsive styling and interactions
  (white-paper styles are the commented block at the end of `site.css`).
- `assets/figs/amd/`: actual modeled geometry and analysis exports for the AMD study.
  Files suffixed `_50um` are copied verbatim from the study's 50 um runs in
  `chipletTherm/3dblox_results/results_amd_mi350_workload_study/high_resolution/`, which
  are the runs WP-102 section 3 tabulates. The unsuffixed `gpu_3d_temperature.png`,
  `memory_3d_temperature.png`, `top_temperature.png` and `gpu_xz_temperature.png` are the
  earlier 250 um exports from that study's `report/assets/`. (Until WP-105 the cover visual
  used `structure_3d_cutaway.png` + `gpu_3d_temperature.png`; it now uses `assets/figs/oam/`.)
  `structure_3d_cutaway_labeled.png` replaces the original `structure_3d_cutaway.png` in
  WP-102: the original used elev=25, which hides the four XCDs on IOD 2 behind the near
  HBM towers. Regenerate it with the study's `render_cutaway_labeled.py`.
- `assets/figs/apr/`: WP-104 figures, from the 10 µm power-map study in
  `chipletTherm/3dblox_results/results_amd_mi350_workload_study/gpu_power_maps_10um/`.
  `apr_20um_overview.png` is made by that folder's `plot_apr_overview.py` from the
  20 µm every-XCD run (scratchpad `g_lr20_all` + `gwin20_all.pkl`; full package + XCD_6 only). The other three are copied
  byte-for-byte from the internal APR Solver artifact (Example 3): `gpu_power_maps_10um.png`,
  `xcd6_power_map_zoom.png` (v2 maps) and `xcd6_20um.png`. WP-104 is a marketing page: it shows
  the high-resolution result, wall time and memory only, and no coarse-grid or alternative runs.
- `assets/figs/oam/`: WP-105 figures and the cover visual, from
  `chipletTherm/examples/ecad_assembly_demo` (synthetic layouts by `make_layouts.py`). Rendered by
  `plot_3d_levels.py run_b_apr_off/fem3d_results.npz` (the APR 50 µm run) with `--clean --title ''`
  (`oam_structure_3d.png`, `oam_temperature_3d.png`) and `--hero` (`oam_hero_structure.png`,
  `oam_hero_temperature.png`), then cropped to content with 16 px padding.
- `assets/figs/risc/`: WP-106 figures, from the internal RISC_CORE thermal study (DEF layout + per-cell
  power, APR 1 µm / 0.2 µm / 50 nm runs). Rendered headless from the study's plotly 3D viewer with a
  white-background overlay (close-up, structure window and whole-die views), then cropped to content with
  32 px padding and resized to 1800 px wide. The layout shown is derived from an academic standard-cell
  library; confirm it may be shown publicly before pushing.

Open `index.html` directly in a browser. No development server is required.
Legacy `#features`, `#technology`, `#results` and `#agentic` entry links on the main page
redirect to the corresponding area or paper. Existing figure/video files are retained.

## Editorial boundaries

The pages describe capabilities and use cases, not proprietary solver implementation.
They intentionally do not claim universal geometry-format support, unattended recovery
of missing physical inputs, sign-off certification, or measured AMD-silicon accuracy.
The numerical workflow includes FEM verification. Do not relabel the AMD sampling study
as a FEM grid-convergence benchmark: its saved runs use the fast numerical solver.

### AMD resolution evidence

Source in the sibling analysis repository:
`chipletTherm/3dblox_results/results_amd_mi350_workload_study/high_resolution/comparison_summary.json`.
Paired coarse runs: `report/runs/gpu_intensive_nonuniform_m48_z2` and
`hbm_spatial_investigation/localized_unequal_m48_z2`. Paired fine runs:
`high_resolution/{gpu,memory}_high_m48_z2`.

Both workloads use 950 W and 25 C ambient. Compare 250 um and 50 um sampling
with unchanged solver representation and Z sampling. Fine fields are interpolated
to coarse cell centers over finite occupied overlaps.

| Workload | Peak difference | Maximum sampled-volume difference | Normalized maximum |
| --- | --- | --- | --- |
| GPU intensive | 0.0233638 K | 0.0382464 K | 0.0443441% |
| Memory intensive | 0.0053753 K | 0.0441014 K | 0.0511788% |

Normalization is maximum sampled absolute difference divided by the fine-grid peak
rise above ambient. It is not local relative error, experimental error, or a GCI band.
The 25x claim refers to X-Y sample count, not runtime. The latest automatic-convergence
results shown in the other AMD exports are separate from this fixed-setting comparison.

### GNN evidence

The existing site's package-specific benchmark is retained under the product name
ThermStack-GNN: 360 unseen power/cooling combinations, 0.32 K mean field RMSE,
10.6 ms per query, and 207x query-time speedup against its FEM reference.
Training/data-generation cost is excluded. These numbers are not AMD-case results
or a claim of accuracy on unseen geometries.

## Publishing and maintenance

Retain this repository's existing GitHub Pages deployment configuration. Publish
only after review; editing the local site does not publish it. Keep both pages,
the shared assets, and the referenced media together.

Use accessible image expansion, keyboard-operable view tabs, mobile navigation
and native video controls. All numerical claims should retain their adjacent scope.
Icons are inlined from Lucide (ISC license in `assets/lucide-LICENSE`).

Contact: noveetyai@noveetymanagement.com
