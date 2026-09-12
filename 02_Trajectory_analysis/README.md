# 02 — Root / CSC cluster and trajectory

**Paper Methods:** Statistical analyses of chromatin opening; Trajectory Analysis.

## Purpose

This is the step that **finds the root / CSC cluster** and then orders cells along a trajectory.

1. Count accessible ATAC peaks per cell
2. Compare clusters (ANOVA + Tukey HSD). The cluster with the highest global accessibility is the candidate root / CSC (cluster 6 in Patient 1)
3. Run PAGA and diffusion pseudotime from a cell inside that cluster
4. Repeat the accessibility comparison on enhancers, promoters, and repeat families (LTR / DNA / LINE / SINE)
5. Write the `*_Trajectory.h5ad` object

The high-accessibility cluster identified using the number of accessible peaks per cell was the same as that identified using total ATAC-seq counts per cell, as reported in the Methods.

## Notebook

- `Trajectory_Patient1_03H096.ipynb`

## Typical inputs

- Cleaned MultiVI object from folder `01` (`Cluster_Final`, ForceAtlas coordinates)
- Peak annotation table: `peak_annotation/teapb2_peaks_annotated.tsv`

## Typical outputs

- `trajectory_objects/MultiVI_Patient1_Relapse_03h096_Trimodal_Cleaned_Trajectory.h5ad`
