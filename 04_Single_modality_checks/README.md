# 04 — Single-modality checks (scVI / PeakVI / CytoVI)

**Paper Methods:** Cross-modal benchmarking of root-cell structure.

## Purpose

Ask whether RNA alone, ATAC alone or protein alone recovers the root / CSC organization seen in MultiVI. These are **benchmarking** analyses, not the primary paper embedding and **not** trajectory analysis.

## What this step creates

- `ScVI_PB2.h5ad`, `PeakVI_PB2.h5ad`, `PB2_CytoVI_adata.h5ad` (Patient 1 example)
- Recovery statistics tables / plots (Extended Data Fig. 7; `Figures/Extended_Data_Figure_7/`)

## Typical inputs

- Same starting multimodal object as MultiVI / TotalVI
- Cleaned MultiVI labels for fair comparison

## Notebooks

Train the unimodal models:

- `scVI_Patient1_03H096.ipynb`
- `PeakVI_Patient1_03H096.ipynb`
- `CytoVI_Patient1_03H096.ipynb`

Score how well those embeddings recover the trimodal LSC Root structure:

- `LSC_recovery_Patient1_03H096.ipynb`
