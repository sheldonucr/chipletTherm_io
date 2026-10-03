# ThermStack website

Customer-facing product overview and feature guide for ThermStack thermal analysis.
Static HTML, CSS and JavaScript; no build step, external fonts or runtime dependencies.

## Information architecture

The site has three areas: **Features & capabilities**, **White papers** and **Contact**.

- `index.html`: the short main page — cover, capability highlights, workflow, the white-paper
  index (`#whitepapers`) and the contact band. It holds no technical figures or tables.
- `whitepapers/wp-1xx-*.html`: the white-paper series. All technical details, benchmarks,
  figures, tables and scope notes live here, each figure/table in exactly one paper:
  - `wp-101-numerical-engine.html` — numerical workflow, model inputs/3Dblox, transient analysis.
  - `wp-102-amd-resolution-study.html` — AMD-inspired study: workloads, 250/50 µm comparison,
    interactive result views.
  - `wp-103-gnn-surrogate.html` — ThermStack-GNN benchmark and usage scope.
  New papers follow the same template: `paper-meta` header, `paper-abstract`, numbered
  `feature-section` blocks, a closing scope section and the series `paper-nav`.
- `features.html`: redirect stub only — its content moved into the white papers; old anchors
  (`#numerical`, `#amd-study`, `#gnn`, …) forward to the right paper.
- `assets/site.css` and `assets/site.js`: shared responsive styling and interactions
  (white-paper styles are the commented block at the end of `site.css`).
- `assets/figs/amd/`: actual modeled geometry and analysis exports for the AMD study.
  Files suffixed `_50um` are copied verbatim from the study's 50 um runs in
  `chipletTherm/3dblox_results/results_amd_mi350_workload_study/high_resolution/`, which
  are the runs WP-102 section 3 tabulates. The unsuffixed `gpu_3d_temperature.png`,
  `memory_3d_temperature.png`, `top_temperature.png` and `gpu_xz_temperature.png` are the
  earlier 250 um exports from that study's `report/assets/`; `index.html`'s cover visual
  still uses the unsuffixed 3D render.
  `structure_3d_cutaway_labeled.png` replaces the original `structure_3d_cutaway.png` in
  WP-102: the original used elev=25, which hides the four XCDs on IOD 2 behind the near
  HBM towers. Regenerate it with the study's `render_cutaway_labeled.py`.

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
