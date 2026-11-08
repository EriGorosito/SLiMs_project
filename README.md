# Protein language model embeddings and sequence–affinity relationships in LxCxE–pRb interactions

University project. This repository contains the code that accompanies the progress report
(`docs/`, written in Spanish). It studies whether the representations learned by protein language models
(pLMs: ESM-2, ProtT5, Ankh) capture how point mutations in LxCxE-motif peptides change their binding to the
retinoblastoma protein (pRb), using deep mutational scanning (DMS) data measured with SIMBA.

Five LxCxE peptides (18 residues each) are analysed: three human (`ARI4A_HUMAN`, `KDM5A_HUMAN`,
`HDAC1_HUMAN`) and two viral (`VE7_HPV16`, `E1A_ADE05_LxCxE`), each with 14 mutated positions (2–15) × 19
substitutions.

## Repository structure

```
├── notebooks/        Analysis, to be run in numerical order
├── data/
│   ├── raw/          Input files (not versioned, see data/README.md)
│   └── processed/    Tables shared between notebooks (generated)
├── results/          Result tables (CSV)
├── figures/          Saved figures
└── docs/             Progress report
```

## Notebooks

Each notebook reads what the previous ones saved in `data/processed/`; **only notebook 01 touches the raw data**.

| # | Notebook | Report section | Main output |
|---|---|---|---|
| 01 | `01_data_preparation` | – | `df_tp.csv`, `df_dms.csv` (ΔZ-scores) |
| 02 | `02_dms_mutation_heatmaps` | 1. Mutational characterisation | DMS heatmaps |
| 03 | `03_plm_embedding_extraction` ⚡ GPU | 2. Embeddings and distances | `wt_embeddings.npz`, `mutant_embedding_distances.csv` |
| 04 | `04_protein_embedding_similarity` | 2. Global similarity between proteins | similarity matrices |
| 05 | `05_embedding_affinity_correlation` | 3. Embedding distance vs. affinity | Pearson r per position |
| 06 | `06_regression_ols_ridge_logo` | 4. OLS / Ridge, leave-one-protein-out | metrics and coefficients |
| 07 | `07_dms_pca` | 5. PCA of mutational profiles | PCA heatmaps and loadings |

Dependency graph: `01 → 02`, `01 → 03 → 04`, `01 + 03 → 05`, `01 + 03 → 06`, `01 → 07`.

## Reproducing the analysis

1. Put the raw files described in [`data/README.md`](data/README.md) in `data/raw/`.
2. Install the dependencies: `pip install -r requirements.txt`.
3. Run the notebooks from the `notebooks/` folder in order (01 → 07).

### Google Colab

Notebook 03 needs a GPU, so it is convenient to run it in Colab (*Runtime → Change runtime type → GPU*):

1. Copy the project folder to Google Drive (e.g. `MyDrive/plm-embeddings-lxcxe-prb`) including `data/raw/`.
2. Open the notebook from Drive. The first cell mounts Drive and uses that folder as `PROJECT_DIR`
   (edit the path in the cell if you used a different name).
3. Outputs are written back to Drive, so the other notebooks can then be run locally or in Colab.

## Methods in brief

* **ΔZ-score:** `Z_mutant − Z_WT` from the SIMBA Z-scores (`C_Pm-Rb` construct, main analysis).
* **Embeddings:** each peptide is embedded inside its full protein sequence; only the peptide window is kept.
* **Distances to the WT:** local `d_p = ‖e_mut,p − e_WT,p‖₂`, global `D = ‖E_mut − E_WT‖_F`.
* **Correlation:** Pearson r between embedding distance and ΔZ across the 19 mutations of each position.
* **Regression:** OLS and Ridge on the 14 positional distances, leave-one-protein-out validation with
  scaling and penalty selection fitted on the training proteins only.
* **PCA:** per-protein position × amino-acid ΔZ matrices.

## Models

| Model | Source |
|---|---|
| ESM-2 (650M) | Lin et al., *Science* 2023 — `facebook/esm2_t33_650M_UR50D` |
| ProtT5-XL-UniRef50 | Elnaggar et al., *IEEE TPAMI* 2022 — `Rostlab/prot_t5_xl_uniref50` |
| Ankh base | Elnaggar et al., 2023 — `ElnaggarLab/ankh-base` |

Experimental data: SIMBA peptide-phage display screens (see the report for references).

## License

To be defined.
