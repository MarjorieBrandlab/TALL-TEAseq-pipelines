# Figure 1

**Figure title:** Trimodal single-cell profiling identifies a chromatin-accessible cell state within T-ALL tumors.

Patients **1** (03H096 / PB2) and **2** (08H125 / PB3). Other samples: Extended Data Fig. 5.

| Panel | Paper | Code |
|---|---|---|
| a | Experimental / MultiVI workflow | Schematic; not drawn here |
| b | ForceAtlas2 of the MultiVI latent space, Leiden clusters. Red arrows = high-accessibility cluster | `plot.ipynb` (`muon.pl.embedding` on `X_draw_graph_fa`) |
| c | Mean ATAC-seq counts per cell ± 95% CI; one-way ANOVA; red arrow = max cluster | `plot.ipynb` (`plot_cluster_openness_with_stats`) |
| d | Same ForceAtlas (`X_draw_graph_fa`), PAGA/DPT with that cluster as root (T1 / T2 terminals) | `plot.ipynb` |

Cluster and DPT maps use the same call (`muon.pl.embedding`, `basis="X_draw_graph_fa"`, `size=100`).

**Objects used for the current paper figures**

| Patient | File under `PATH_TO_DATA` | Root cluster |
|---|---|---|
| 1 | `TALL_Website/Object/PB2/Teaseq_Multi_VI_PB2_Cleaned.h5ad` | 6 |
| 2 | `New_Analysis/MultiVI/PB3/Teaseq_Multi_VI_PB3_Cleaned.h5ad` | 5 |

`PATH_TO_DATA` is the `TALL_Review` root (the folder that contains `New_Analysis/` and `TALL_Website/`).

ForceAtlas size is fixed: `dpi=100`, `figsize=(6.4, 4.8)`, `size=100`. Extended Data Fig. 5 uses the same call.
