# 03 — TotalVI (RNA + surface protein)

**Paper Methods:** Trimodal integrated analyses / differential expression.

## Purpose

Train TotalVI to obtain **denoised surface-protein abundances** used to design FACS gates for root vs terminal populations, and to support RNA / protein / TF differential analyses.

## What this step creates

- `TotalVI_*_TEAseq_CLEAN_TF.h5mu` (or equivalent patient MuData)
- Denoised protein / RNA layers and DE tables used in Figure 2 and supplements

## Typical inputs

- Raw multimodal MuData (protein + RNA)
- Cleaned MultiVI object (to transfer cluster / trajectory labels onto the TotalVI object)
- Optional TF gene list (`TFs_Ensembl_v_1.01.txt`) for TF-focused DE

## Notebook

- `TotalVI_Patient1_03H096.ipynb`

## Known issue with the dot plots in this notebook

Three cells call `sc.pl.dotplot` / `sc.pl.matrixplot` with **both** `standard_scale="var"` and `swap_axes=True`. In **scanpy 1.10.4** (the version in the Methods) that combination mis-maps the dot colours: the plotted colours do not match the object's own `dot_color_df`, so the highest-expressing cluster can render pale. It is correct again in scanpy 1.12.1.

Anything reused for a figure should be re-rendered. The pattern that is safe on both versions is to scale first and hand the finished frames over:

```python
scaled = sc.pl.DotPlot(adata, genes, groupby=key, use_raw=False, standard_scale="var")
dp = sc.pl.DotPlot(adata, genes, groupby=key, use_raw=False,
                   dot_color_df=scaled.dot_color_df, dot_size_df=scaled.dot_size_df)
dp.swap_axes().style(cmap="Reds").show()
```

The TF dot plots in `Figures/Figure_3/plot_TF.ipynb` and `Figures/Extended_Data_Figure_12/plot_TF.ipynb` build their grids directly in matplotlib and are unaffected.
