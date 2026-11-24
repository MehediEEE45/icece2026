# Censoring-aware cross-cell benchmark for Li-ion RUL and SOC (NASA PCoE)

Code, results and manuscript for *Censoring-Aware Remaining Useful Life Prediction of
Lithium-Ion Batteries with Conformal Uncertainty: A Cross-Cell Benchmark* (ICECE 2026).

All code is in Jupyter notebooks. Every notebook is committed **with its outputs**, so the
numbers and figures in the paper can be checked without running anything.

## Notebooks

Run them in order from the `notebooks/` folder. Each one loads the shared definitions with
`%run ./00_common.ipynb`.

| Notebook | What it does | Paper |
|---|---|---|
| `00_common.ipynb` | paths, data loading, screening, causal features, models, metrics, conformal and SOC helpers | — |
| `01_rul_benchmark.ipynb` | screening of all 34 cells, impedance missingness, feature correlation matrix, centered vs causal features, impedance ablation, 5-seed spread, Wilcoxon tests | Fig. 1, Fig. 2, Sec. III |
| `02_nested_selection.ipynb` | nested leave-one-cell-out configuration selection, Table II, Wilcoxon tests | Table II, Fig. 4 |
| `03_controls_conformal_deploy.ipynb` | same-cell controls (leaky, matched, LOBO), split and CV+ conformal intervals, deployability | Fig. 5, Fig. 8, Table III |
| `04_lite_tft.ipynb` | lite-TFT with early stopping (set `FIXED_EPOCHS = 60` for the fixed-epoch protocol), int8 size, activation memory | Table II, Sec. V-F |
| `05_soc_benchmark.ipynb` | SOC benchmark, SOH-shortcut ablation, coulomb-counting and OCV baselines | Figs. 6–7, Sec. V-C/D |
| `06_figures.ipynb` | every figure, drawn at its printed size | Figs. 1–8 |

Results are written to `results/` (CSV) and figures to `figs/` (600 dpi PNG).

## Reproduce

```bash
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/cpu
cd notebooks
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=-1 0[1-6]_*.ipynb
```

Notebooks 01–04 need only `data/metadata.csv`. Notebook 05, and the trace figure in 06, also
need the raw discharge waveforms of B0005, B0006, B0007, B0018, B0042 and B0054 from the
NASA PCoE battery data set. Put them under `organized/<cell>/discharge/<file>.csv`, with file
names matching the `filename` column of `metadata.csv`, or point the `PCOE_ORGANIZED`
environment variable at that folder. The raw files are not included because of their size.

## Protocol in brief

- All 34 PCoE cells are screened. The cohort is 10 cells at 80% SOH end of life.
- The EOL label uses a centered 5-point median of SOH, computed offline.
- Every model feature uses a trailing (causal) median, so no feature looks beyond cycle k.
- Evaluation is leave-one-battery-out.
- Per model, an inner leave-one-cell-out loop picks one of four configurations: with or
  without the cycle-age feature, and target RUL or RUL/k. The held-out cell never enters
  this choice.
- Hyperparameters are fixed defaults (`results/hyperparameters.csv`).

## Repository layout

```
data/metadata.csv   NASA PCoE metadata (capacity and impedance per cycle)
notebooks/          00–06, executed, with outputs
results/            every CSV the paper's numbers come from
figs/               figures
paper/              manuscript source and PDF
requirements.txt    exact package versions used
```

## Data

NASA Ames Prognostics Data Repository, Battery Data Set (B. Saha and K. Goebel, 2007).
