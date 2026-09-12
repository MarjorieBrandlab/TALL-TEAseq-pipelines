# Figures

Plotting notebooks for *TEASeq-Paper-21.docx*. Numbering is the caption list in that Word file.

Each folder is **one paper figure**. Open the notebook and set PATH_TO_DATA (or PATH_TO_ATAC for Extended Data Fig. 8) to the location of the corresponding saved analysis objects before running it. The notebook loads the saved analysis object and draws the panel. It does not retrain MultiVI / TotalVI / SCENIC+.

Pipelines `01`–`05` are unchanged. This directory only plots.

## How to run a notebook

1. Point `DATA = Path("PATH_TO_DATA")` at the **`TALL_Review` root** (the folder that contains `New_Analysis/` and `TALL_Website/`). ForceAtlas maps must come from the **current cleaned MultiVI objects** listed below — not from `Ted_Objects_for_Pipelines/.../trajectories/` (those are the 2025-02 layouts, no longer in the paper).
2. Run the notebook from **its own folder** (relative paths and preview PNGs assume that).
3. Scanpy / muon versions in the Methods: scanpy 1.10.4, muon 0.1.7, mudata 0.3.1.

## ForceAtlas objects used in the paper

| Paper | File |
|---|---|
| Patient 1 (PB2) | `TALL_Website/Object/PB2/Teaseq_Multi_VI_PB2_Cleaned.h5ad` (no `New_Analysis/MultiVI/PB2` folder) |
| Patient 2 (PB3) | `New_Analysis/MultiVI/PB3/Teaseq_Multi_VI_PB3_Cleaned.h5ad` |
| Patient 1′ (BM4) | `New_Analysis/Scenicplus/New_BM4/Teaseq_Multi_VI_BM4_Cleaned.h5ad` |
| Patient 3 (BM5) | `New_Analysis/Scenicplus/New_BM5/Teaseq_Multi_VI_BM5_Cleaned.h5ad` |
| Patient 4 (BM1) | `New_Analysis/Scenicplus/New_BM1/Teaseq_Multi_VI_BM1_Cleaned.h5ad` |
| Patient 5 (BM3) | `New_Analysis/Scenicplus/New_BM3/Teaseq_Multi_VI_BM3_Cleaned.h5ad` |

ED5 loads these MultiVI Updated/Final objects. Do not use `New_Analysis/Scenicplus` or Ted trajectory files.

Patient labels:

| Paper | Sample ID | Lab label |
|---|---|---|
| Patient 1 (relapse; main example) | 03H096 | PB2 |
| Patient 1′ (diagnosis) | 03H005 | BM4 |
| Patient 2 | 08H125 | PB3 |
| Patient 3 | 19H007 | BM5 |
| Patient 4 | 05H080 | BM1 |
| Patient 5 | 21H048 | BM3 |

## Plot size (do not change per panel)

ForceAtlas maps must look the same size in every figure. All embedding notebooks use:

```python
sc.set_figure_params(dpi=100, fontsize=12, frameon=False, figsize=(6.4, 4.8))
FA_SIZE = 100
```

and `muon.pl.embedding(..., basis="X_draw_graph_fa", size=FA_SIZE, ncols=1)`.

Raising `dpi` or shrinking `figsize` makes the points look bigger on GitHub even when `size=100`. Do not do that.

Other panel types, also fixed:

| Panel type | Size |
|---|---|
| ATAC peaks/cell bars (Fig. 1c, ED5) | `figsize=(10, 6)` |
| QC metric grids (ED2) | `figsize=(10, 6)` |
| ADT gate scatters (Fig. 2) | `dpi=300`, scatter `size=100` |
| ELDA curves (Fig. 2h,i) | `figsize=(5.8, 4.8)`, `dpi=300` |

## Main figures

| Paper | Title | Notebook |
|---|---|---|
| **Figure 1** | Trimodal single-cell profiling identifies a chromatin-accessible cell state within T-ALL tumors. | `Figure_1/plot.ipynb` — ForceAtlas Leiden, peaks/cell ANOVA, PAGA/DPT. Patients 1 and 2. |
| **Figure 2** | Chromatin-accessible Root cells represent functional CSCs in T-ALL. | No gating here. `Figure_2/plot_elda.ipynb` (primary / secondary). STORM is experimental, not here. |
| **Figure 3** | Chromatin-accessible CSCs activate stem/progenitor-associated transcription factors and enhancers. | `Figure_3/plot_TF.ipynb` (DE TF counts + top-50 dots, Patient 1). `plot_FA_genes.ipynb` (ERG, RUNX1, MYB on ForceAtlas). Enhancer tracks are bigWigs, not in Git. |
| **Figure 4** | CSCs activate patient-specific regulatory networks built around stem/progenitor-associated transcription factors. | `Figure_4/plot_eRegulon_heatmap.ipynb`. Patient 1. Other patients: ED14. |
| **Figure 5** | CSC-network regulators RUNX1 and ERG are required for the leukemia-initiating activity of CSCs in vivo. | Wet lab (qPCR + transplant). No TEA-seq notebook. |

## Extended Data

| Paper | Title | Notebook |
|---|---|---|
| **ED 1** | ATAC quality-control metrics for TEA-seq libraries. | Cell Ranger ARC library QC. No notebook in this repo. |
| **ED 2** | Cell-level quality control metrics for TEA-seq libraries. | `Extended_Data_Figure_2/plot.ipynb` — RNA / ADT violins from `Metrics/<lab>/metrics_paper_final.csv`. |
| **ED 3** | Identification of leukemic cells in primary T-ALL samples. | `Extended_Data_Figure_3/plot.ipynb` — clinical ADT heatmap only. |
| **ED 4** | Molecular subtype annotation using the Polonen et al. reference cohort. | R / Seurat. No notebook in this repo. |
| **ED 5** | Chromatin-accessible Root states are detected across additional primary T-ALL samples. | `Extended_Data_Figure_5/plot.ipynb` — same three panels as Figure 1b–d for Patients 1′, 3, 4, 5. |
| **ED 6** | Increased chromatin accessibility in the Root clusters is distributed across genomic element classes. | `Extended_Data_Figure_6/assemble_peak_classes.ipynb` — 6×6 ATAC-seq counts/cell by class. |
| **ED 7** | Contribution of individual modalities to the structure of the trimodal-defined Root cluster. | `Extended_Data_Figure_7/plot.ipynb` — recovery heatmaps only (all six samples). |
| **ED 8** | 40-patient public scATAC (Xu et al.). | `Extended_Data_Figure_8/export_naked_layout.ipynb`. Set `PATH_TO_ATAC`. |
| **ED 9** | TEA-seq-guided sorting strategy for Patient 1 Root cells. | No notebook (see paper / Supplementary Note 3). |
| **ED 10** | TEA-seq-guided sorting strategies for Patient 1 terminal-state cells. | No notebook (see paper / Supplementary Note 3). |
| **ED 11** | TEA-seq-guided sorting strategies for Patient 2 Root and terminal-state cells. | No notebook (see paper / Supplementary Note 3). |
| **ED 12** | CSC/Root clusters express a broader transcription factor repertoire across T-ALL samples. | `Extended_Data_Figure_12/plot_TF.ipynb` — DE-TF bar plots (a) + top-50 dots (b); no colorbar/legend. |
| **ED 13** | Cell fate probabilities across leukemic trajectories in Patient 1. | `Extended_Data_Figure_13/plot.ipynb` — two FA maps, MIRA T1 / T2 fate scores (P1). |
| **ED 14** | Patient-specific gene regulatory networks across CSC/Root and terminal leukemic states. | `Extended_Data_Figure_14/plot_heatmap.ipynb`. |
| **ED 15** | Enrichment of a T-cell pre-commitment TF module in CSC-associated GRNs. | `Extended_Data_Figure_15/plot.ipynb`. |

## Not deposited in this repository


- Sorting / gate panels (Fig. 2b,c; Extended Data Figs. 9–11)
- Fig. 4c–e network views and Fig. 4f GRN-overlap plot
- ED14 network graph layouts (heatmap notebook only)
- Schematics (Fig. 1a, 2a, 4a, 5a), STORM (Fig. 2d–g), enhancer tracks, knockdown plots, Polonen UMAP (ED4), Fig. 3d lineage tree

Interactive GRNs: https://tall.systemsbiology.net
