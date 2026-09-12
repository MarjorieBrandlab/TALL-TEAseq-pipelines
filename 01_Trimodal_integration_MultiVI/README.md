# 01 — Trimodal MultiVI integration

**Paper Methods:** Trimodal integrated analyses

## Purpose

Generate the joint trimodal representation used in Fig. 1 and downstream analyses by integrating scRNA-seq, scATAC-seq, and cell-surface protein data with MultiVI. This notebook performs RNA, ATAC, and protein quality control, including removal of IgG isotype-control ADTs, and aligns the three modalities on shared cell barcodes.

## Analysis overview

- The Patient 1 example uses a Leiden resolution of 0.8. As described in the Methods, the resolution was adjusted across samples after examining the expression of key proteins or genes.
- The high-accessibility cluster is identified by comparing the number of accessible ATAC-seq counts per cell across leukemic Leiden clusters. Trajectory analysis in `02` is then used to assess its root-like position.
- CellTypist annotations generated at the end of this notebook contribute to the complementary strategy used to distinguish leukemic blasts from non-leukemic populations. Leukemic identity was further assessed using deletion of `MTAP` and `CDKN2A` in the chromatin-accessibility data, loss of their expression in the transcriptomic data, and clinical cell-surface marker profiles (Extended Data Fig. 3). T-ALL subtype assignments based on genomic rearrangements and comparison with the Polonen et al. reference cohort are shown in Extended Data Fig. 4.

## What this step creates

- Cleaned MultiVI AnnData objects for each sample, for example:
  - `MultiVI_Patient1_Relapse_03H096_Trimodal_Cleaned.h5ad` (Patient 1; PB2)
- MultiVI latent representation (`X_multivi`)
- Neighbourhood graph
- Leiden cluster assignments
- ForceAtlas2 coordinates

## Typical inputs

- Preprocessed multimodal MuData assembled from Cell Ranger ARC and CITE-seq-Count outputs, for example `Teaseq_PB2.h5mu`
- RNA, ATAC, and protein matrices with consistent cell-barcode identifiers

## Notebook

- `MultiVI_Patient1_03H096.ipynb`

**Note:** The cleaned MultiVI objects generated in this step are reloaded by later notebooks for downstream analyses.
