# T-ALL TEA-seq analysis and figure code

Analysis notebooks and figure-generating code supporting the T-ALL TEA-seq manuscript:

**Increased global chromatin accessibility identifies leukemic stem cells**

TEA-seq simultaneously measures three modalities in the same cell:

1. gene expression (RNA)
2. chromatin accessibility (ATAC)
3. 164 cell-surface proteins (ADT / TotalSeq)

The folders broadly follow the **Data Analyses** section of the Methods.
Example notebooks use **Patient 1 (sample 03H096; laboratory label PB2)**. The same workflow was applied to the other patient samples.

---

## Repository scope

This repository contains the analysis and&#x20;
figure-generating code supporting the manuscript.
Readers can examine how the analyses were performed by browsing the&#x20;
notebooks and figure-generating code without rerunning the&#x20;
computationally intensive model-training steps.

**Intermediate files and paths**

- This repository does not include the large intermediate analysis objects.
- The notebooks create those objects (cleaned MultiVI objects, trajectory objects, totalVI MuData, Root annotations, SCENIC+ outputs, etc.).
- When a notebook uses `read_h5ad(...)` or `muon.read(...)`   to load an intermediate file, that file is the saved output of an earlier documented step or an earlier stage of the same notebook.
- Large intermediates (`.h5ad`, `.h5mu`, fragment files, motif DBs) are not stored in Git. Every notebook declares `DATA_DIR = Path("PATH_TO_DATA")` at the top and addresses all files relative to it, so only `DATA_DIR` needs to be set to the local data root.

**Public data (GEO)**

- TEA-seq: **GSE268989**

---

## Sample names used in the paper

In the manuscript, samples are labeled Patient 1, Patient 1′, Patient 2, etc.
Notebooks and paths may still use laboratory labels:

| Paper label | Sample ID | Laboratory label in paths |
| --- | --- | --- |
| Patient 1 (relapse)                                | 03H096 | PB2 |
| Patient 1′ (diagnosis)                             | 03H005 | BM4 |
| Patient 2                                          | 08H125 | PB3 |
| Patient 3                                          | 19H007 | BM5 |
| Patient 4                                          | 05H080 | BM1 |
| Patient 5                                          | 21H048 | BM3 |

Whenever possible, notebooks here are named **Patient1\_03H096**.

Some legacy filenames use `LSC`. In the current manuscript, the corresponding populations are referred to as **Root** populations or **CSCs**.

---

## How to read this repository (quick guide)

If you only have a few minutes, open these four:

1. `01_Trimodal_integration_MultiVI` — main joint embedding used for Fig. 1 (creates cleaned MultiVI objects; RNA, ATAC and protein QC are performed here)
2. `02_Trajectory_analysis` — identification of the high-accessibility Root cluster and trajectory analysis
3. `03_RNA_and_protein_TotalVI` — surface markers used to sort CSCs (creates totalVI MuData)
4. `05_Gene_regulatory_networks` — SCENIC+ networks for CSC/Root and terminal states

Then use the `Figures/` folder for the plotting code.

**Suggested object flow (high level)**

```
GEO data
   → 01 MultiVI (QC filters + integration + Leiden clustering; resolution 0.8 in the Patient 1 example)
                                              → cleaned MultiVI .h5ad
   → 02 Root cluster (accessibility) + PAGA/DPT  → *_Trajectory.h5ad
   → 03 totalVI RNA+protein                   → totalVI .h5mu + DE tables
   → 04 single-modality checks                → scVI / PeakVI / CytoVI embeddings
   → 05 SCENIC+ (cluster): preprocessing notebook, then Snakemake (`config.yaml`)
                                              → scplusmdata.h5mu / eRegulons
   → Figures/                                 → manuscript panels


```

---

# Part A. Analysis pipelines (paper Methods order)

## A1. Trimodal integration with MultiVI (main analysis)

**Paper Methods:** Pre-processing + Trimodal integrated analyses

**What this step does**

This is the **central computational step** of the paper. RNA / ATAC / protein QC and MultiVI training live in the same notebook.

After Cell Ranger / CITE-seq-Count processing, each TEA-seq sample is filtered:

- RNA: require at least 200 detected  genes per cell; retain genes  detected in at least 3 cells; restrict  cells to 350–5,000 detected genes  and less than 40% mitochondrial  content
- ATAC: remove rare peaks (present in less than 0.5% of cells) and non-canonical contigs
- Protein: remove IgG control and non-human ADT features (including Rat, Mouse and Hamster features, where present), then retain barcodes shared with filtered RNA/ATAC

RNA, ATAC and protein are then jointly modelled with **MultiVI** (`scvi-tools`). The trained latent representation is used to:

1. build a nearest-neighbour graph
2. generate Leiden clusters; Patient 1 example uses resolution 0.8
3. project cells with ForceAtlas2 for visualization only
4. inspect ATAC-seq counts per cell to identify the high-accessibility candidate Root cluster (continue in `02`)

**Leukemic blasts and non-leukemic populations were identified using complementary approaches** (Results; Extended Data Fig. 3):

1. CellTypist transcriptomic annotation (this notebook)
2. MTAP/CDKN2A deletion in scATAC plus loss of MTAP/CDKN2A RNA
3. TEA-seq surface markers vs the clinical flow-cytometry report
4. Genomic rearrangements and Polonen et al. AALL0434 subtype projection (Extended Data Fig. 4)

QC metric plots: ATAC library QC is Cell Ranger ARC (Extended Data Fig. 1). Per-cell RNA / ADT violins are `Figures/Extended_Data_Figure_2/`.

**Notebook**

- `01_Trimodal_integration_MultiVI/MultiVI_Patient1_03H096.ipynb`

**Related paper content**

- Fig. 1a–d (workflow, FA maps, ATAC-seq counts per cell and pseudotime)
- Extended Data Figs. 1–5
- Supplementary Table 3, Supplementary Note 1
- Methods text on MultiVI training and clustering

**Figure plotting helpers**

- `Figures/Figure_1/`
- `Figures/Extended_Data_Figure_2/`
- `Figures/Extended_Data_Figure_5/`

---

## A2. Chromatin accessibility across clusters

**Paper Methods:** Statistical analyses of chromatin opening in each patient

**What this step does**

For each patient, total ATAC-seq counts per&#x20;
cell are compared across Leiden clusters by one-way ANOVA, followed by&#x20;
Tukey's HSD test when the overall test is significant. The cluster with&#x20;
the highest global accessibility is designated the candidate Root&#x20;
population.

**Notebooks / figure code**

- `Figures/Figure_1/`
- `Figures/Extended_Data_Figure_5/`
- `Figures/Extended_Data_Figure_6/`

**Related paper content**

- Fig. 1c
- Extended Data Fig. 5
- Extended Data Fig. 6
- Methods ANOVA summaries per patient

---

## A3. Root cluster and trajectory

**Paper Methods:** Statistical analyses of chromatin opening; Trajectory Analysis

**What this step does**

For the Patient 1 example, the Leiden clustering at&#x20;
resolution 0.8 serves as the starting partition for the subsequent&#x20;
Root-cluster analysis. This notebook:

1. counts accessible ATAC peaks per cell
2. compares clusters using one-way ANOVA  followed by Tukey's HSD test —  the highest-accessibility cluster is  the candidate Root population  (cluster 6 in Patient 1)
3. runs PAGA and diffusion pseudotime from that root toward terminal states
4. infers terminal-state fate probabilities with MIRA
5. repeats the accessibility comparison on enhancers, promoters, and repeat families
6. writes the `*_Trajectory.h5ad` object

**Notebook**

- `02_Trajectory_analysis/Trajectory_Patient1_03H096.ipynb`

**Related paper content**

- Fig. 1c,d
- Extended Data Fig. 5
- Extended Data Fig. 6
- Extended Data Fig. 13
- Methods chromatin opening and Trajectory Analysis

---

## A4. totalVI: joint RNA and surface-protein analysis (markers for sorting)

**Paper Methods:** Trimodal integrated analyses; Differential Expression

**What this step does**

totalVI jointly models RNA and protein. It is central to the prospective-isolation strategy because:

- it produces denoised protein abundances used to design FACS gates
- it supports differential protein / RNA / TF analyses across clusters

Typical totalVI workflow in the notebook:

1. QC of RNA and protein modalities
2. select highly variable genes (top 4,000, Seurat v3)
3. train totalVI
4. extract latent representation and denoised expression (`get_normalized_expression`, n\_samples = 25, mean)
5. neighbours, Leiden, ForceAtlas visualization
6. differential expression / abundance for markers and TFs

**Notebook**

- `03_RNA_and_protein_TotalVI/TotalVI_Patient1_03H096.ipynb`

**Related paper content**

- Fig. 2, Extended Data Figs. 9–11 (sorting-strategy design and validation)
- Fig. 3a,b and Extended Data Fig. 12 (transcription-factor differential-expression analyses)
- Supplementary Tables 5, 6, 8, 9
- Methods Trimodal integrated analyses and Differential Expression sections

**Figure plotting helpers**

- `Figures/Figure_3/`
- `Figures/Figure_2/`

---

## A5. Unimodal models (scVI, PeakVI, CytoVI)

**Paper Methods:** Cross-modal benchmarking of root-cell structure

**What this step does**

The paper asks how well the Root population can be recovered from one modality alone. Matched latent spaces are compared:

- MultiVI (trimodal reference)
- scVI (RNA only)
- PeakVI (ATAC only)
- CytoVI (protein only)

Metrics include neighbour purity of Root cells, neighbourhood preservation relative to MultiVI, silhouette separation, separation ratio, and a composite global recovery score. Each individual modality recovers some aspects of the trimodal-defined Root structure, but no single modality fully recapitulates the local and global structure observed in the integrated representation.

**Notebooks** (train the unimodal models)

- `04_Single_modality_checks/scVI_Patient1_03H096.ipynb`
- `04_Single_modality_checks/PeakVI_Patient1_03H096.ipynb`
- `04_Single_modality_checks/CytoVI_Patient1_03H096.ipynb`

**Recovery metrics** (Extended Data Fig. 7; the plotting directory retains a legacy name)

- `04_Single_modality_checks/LSC_recovery_Patient1_03H096.ipynb`
- Plotting copy: `Figures/Supplementary_Figure_4/plot.ipynb`

**Related paper content**

- Extended Data Fig. 7
- Supplementary Note 2

---

## A6. Gene regulatory networks (SCENIC+)

**Paper Methods:** Gene Regulatory Networks (preprocessing → topic modelling → eGRN inference → network visualization → comparative analyses)

This is a multi-stage pipeline. **Both A6.1 and A6.2 are designed to run on a computing cluster** because peak calling, MALLET LDA, motif enrichment and GBM linking are computationally intensive.

### A6.1 SCENIC+ preprocessing (Jupyter, cluster)

Inputs are cleaned MultiVI objects (clusters + ForceAtlas&#x20;
coordinates). The notebook prepares RNA, builds ATAC pseudobulk peaks&#x20;
(MACS2), creates the cisTopic object, and fits topic models. It writes `cistopic_obj.pkl`, `scRNA/adata.h5ad` and `region_sets/` — the three inputs Snakemake needs. Submit this notebook on a cluster.

**Notebook**

- `05_Gene_regulatory_networks/SCENICPLUS_preprocessing_Patient1_03H096.ipynb`

### A6.2 eGRN inference is a Snakemake pipeline (cluster, not Jupyter)

The walkthrough notebook listed below does&#x20;
not perform eRegulon inference; inference is run through the SCENIC+&#x20;
Snakemake workflow. After A6.1, the official SCENIC+ **Snakemake** workflow (Snakefile shipped with the `scenicplus` package) does motif enrichment, TF–region–gene linking, eRegulon assembly and AUCell scoring, and writes `scplusmdata.h5mu`.

On the cluster:

```
scenicplus init_snakemake --out_dir scplus_pipeline
# creates scplus_pipeline/Snakemake/workflow/Snakefile
#     and scplus_pipeline/Snakemake/config/config.yaml  (default — replace it)

cp 05_Gene_regulatory_networks/config.yaml \
   scplus_pipeline/Snakemake/config/config.yaml

cd scplus_pipeline/Snakemake
snakemake --cores 20

```

That job used 20 CPUs. `temp_dir` is HTCondor node scratch (`${_CONDOR_SCRATCH_DIR}`) so intermediate files do not fill shared storage.

**The full config that was submitted** (cluster paths from the Patient 1 / PB2 run):

- `05_Gene_regulatory_networks/config.yaml`

This file specifies the inputs, outputs and analysis parameters. Set `PATH_TO_PROJECT` and `PATH_TO_MOTIF_DB` to the corresponding paths on the computing cluster. A walkthrough of the same commands is in:

- `05_Gene_regulatory_networks/SCENICPLUS_snakemake_Patient1_03H096.ipynb`

### A6.3 Network visualization and comparative GRN figures

Plotting code is under `Figures/Figure_4/` (Patient 1) and `Figures/Extended_Data_Figure_14/` (other patients).

**Related paper content**

- Figure 4
- Extended Data Fig. 14
- Extended Data Fig. 15
- Supplementary Table 10
- Methods comparative IoU / permutation tests and pre-commitment TF module analysis

---

# Part B. Figure notebooks (plotting code)

Plotting notebooks are organized under `Figures/`. They load the completed objects from pipelines `01`–`05` and generate manuscript panels; they do not rerun the upstream analyses.

See `Figures/README.md` for the panel-to-file map and for what is *not* a plotting notebook (schematics, STORM, qPCR, bigWig tracks).

---

# Part C. Suggested reading order

### To understand Figure 1 (trimodal map + Root population)

1. `01_Trimodal_integration_MultiVI/MultiVI_Patient1_03H096.ipynb`
2. `02_Trajectory_analysis/Trajectory_Patient1_03H096.ipynb`
3. `Figures/Figure_1/`

### To understand Figure 2 (sorting + functional CSC validation)

1. `03_RNA_and_protein_TotalVI/TotalVI_Patient1_03H096.ipynb`
2. `Figures/Figure_2/`

### To understand Figs. 3–4 (TF programs and GRNs)

1. `Figures/Figure_3/`
2. `05_Gene_regulatory_networks/SCENICPLUS_preprocessing_Patient1_03H096.ipynb`
3. `05_Gene_regulatory_networks/config.yaml` (SCENIC+ Snakemake pipeline — not a notebook)
4. `Figures/Figure_4/`

### To understand the single-modality benchmarking

1. `04_Single_modality_checks/scVI_Patient1_03H096.ipynb`
2. `04_Single_modality_checks/PeakVI_Patient1_03H096.ipynb`
3. `04_Single_modality_checks/CytoVI_Patient1_03H096.ipynb`
4. `04_Single_modality_checks/LSC_recovery_Patient1_03H096.ipynb`

---

# Part D. Software environment

Core software and versions reported in the Methods:

- Pre-processing: Cell Ranger Arc v2.0.2, Cell Ranger v7.1.0, bcl2fastq v2.20 and CITE-seq-Count v1.4.5
- Integrated and single-modality analyses: `scvi-tools` v1.2 (MultiVI, totalVI, scVI, PeakVI and CytoVI), Scanpy v1.10.4, anndata v0.11.1, muon v0.1.7 and PyTorch v2.0
- Trajectory and fate-probability analyses: MIRA (`mira-multiome` v2.1.1)
- Gene regulatory network analyses:  SCENIC+ v1.0a1, pycisTopic v2.0a0,  pycistarget v1.0a2, mudata v0.3.1,  pandas v2.2.3 and NumPy v1.26.4
- Peak calling and topic modelling: MACS2 v2.2.9.1 and MALLET binary release 202108
- Snakemake for the official SCENIC+ eGRN pipeline

Minimal environment setup for the scvi-tools notebooks:

```
conda create -n tall-teaseq python=3.11
conda activate tall-teaseq
pip install scvi-tools==1.2.0 scanpy==1.10.4 anndata==0.11.1 muon==0.1.7 mudata==0.3.1 matplotlib seaborn pandas

```

For SCENIC+, follow the official install guide: [https://scenicplus.readthedocs.io/](https://scenicplus.readthedocs.io/)

Large intermediate files (`.h5ad`, `.h5mu`, fragment files, motif databases) are **not** stored in this GitHub repository.

- To **understand** the Methods implementation, read the notebooks and their documented analysis steps.
- To **re-run** a step: use objects generated by the preceding folders or regenerate the required outputs from the data deposited in GEO.
- Notebooks were written as working analysis documents (train → save →  reload). Reloading a previously  written object is expected, not a  substitute for the creation step.
- Every notebook opens with a `DATA_DIR = Path("PATH_TO_DATA")` placeholder and an `OUT_DIR` for the files it writes. Set `DATA_DIR` to your local data root before re-running; all input and output paths are expressed relative to it.

---

# Part E. Interactive GRN browser

Interactive gene regulatory networks for CSC/Root and terminal-state programs from each patient can be explored at [https://tall.systemsbiology.net](https://tall.systemsbiology.net). During peer review, access to the site requires a username and password, provided separately to reviewers. Access will be unrestricted upon publication.

The GRNs were inferred using SCENIC+ as described in Part A6.

---

## Contact

Questions about the code or analyses can be directed to the corresponding authors of the manuscript.
