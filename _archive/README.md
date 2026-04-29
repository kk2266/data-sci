# Archive

Files that are kept for reference but are not part of the live pipeline. Nothing in this folder is read by any of the notebooks in [`../notebook2/`](../notebook2). It is safe to delete the whole folder if disk space is tight.

## What's here

```
_archive/
├── docs/                    ← course / brief documents
│   └── empiricalProject_2026.pdf
│
├── figures_old/             ← chart PNGs from earlier iterations of the project
│   ├── chart_3_scatter.png
│   ├── chart_4_heatmap.png
│   ├── chart_5_feature_importance.png
│   ├── chart_5b_feature_importance_per_theme.png
│   ├── chart_6_pred_vs_actual.png
│   ├── chart_7_module_sizes.png
│   └── chart_8_robustness_r_vs_t.png
│
├── module_bank_old/         ← snapshot of the scrape before the STEM re-fetch
│   └── modules_details_v1backup.csv
│
└── nss_xlsx/                ← raw Excel downloads from the OfS
    └── NSS{23,24,25}_10007792{,_- Copy}.xlsx
```

## Why each item is here

- **`figures_old/`** — these PNGs came from earlier numberings of the chart pipeline (when there were eight charts in the post rather than four plus diagnostics). The current notebook 06 writes only `chart_1_…` through `chart_4_…` and four `diag_*` files into `output/figures/`. The old PNGs are kept so the iteration history is recoverable, but they don't correspond to anything in the live blog or code.
- **`module_bank_old/modules_details_v1backup.csv`** — backup of the raw scrape that notebook 02.5 wrote before overwriting `data/raw/module_bank/modules_details.csv` with the STEM-corrected version. Useful for diffing the two scrape passes; not used as input by any notebook.
- **`nss_xlsx/`** — original Excel files downloaded from the Office for Students. Notebook 04 reads only the `_r.csv` and `_t.csv` exports of these files (which live in `data/raw/nss/`), so the source workbooks aren't needed at runtime. Kept here as the canonical source in case anyone needs to verify the CSV exports.
- **`docs/empiricalProject_2026.pdf`** — the course assignment brief. Not part of the project output and not for redistribution.
