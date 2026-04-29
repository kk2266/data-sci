# How Exeter assesses its students

A data-driven blog post for **BEE2041 — Data Science in Economics** (April 2026).

Six years of every undergraduate module Exeter has published, joined to three years of National Student Survey results, used to ask three questions:

1. How does each Exeter department mix coursework with exams?
2. Has it changed over time?
3. Does any of it line up with how satisfied students say they are?

The blog post lives in [`blog.html`](blog.html). The notebooks in [`notebook2/`](notebook2) are the code that produced everything in it.

---

## Read the blog

- **Live version:** *(once published, place the GitHub Pages URL here, e.g. `https://YOUR-USERNAME.github.io/REPO/blog.html`)*
- **Local version:** open [`blog.html`](blog.html) in any browser. The four chart PNGs in `output/figures/` are referenced by relative path, so the page renders without any server.

---

## Quick replication

```bash
git clone https://github.com/YOUR-USERNAME/REPO.git
cd REPO
python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

Then open the notebooks in [`notebook2/`](notebook2) and run them in numerical order. The cleaning, NSS processing, merging and analysis notebooks (03 → 06) take less than two minutes end-to-end on a normal laptop. The scraping notebooks (01, 02, 02.5) are optional because the scraped CSVs are already committed to the repo, see *Reproducing from scratch* below.

**Python version:** 3.10 or newer. Tested on 3.10.12 and 3.12.7.

---

## What's in this folder

```
files/
├── blog.html                       ← the blog post, open in any browser
├── README.md                       ← you are here
├── requirements.txt                ← Python packages used by the notebooks
│
├── notebook2/                      ← the seven Jupyter notebooks
│   ├── 01_scrape_module_index3.ipynb
│   ├── 02_scrape_module_details.ipynb
│   ├── 02_5_rescrape_stem_modules.ipynb     ← STEM page-layout fix, run once
│   ├── 03_clean_module_data.ipynb
│   ├── 04_process_nss3.ipynb
│   ├── 05_merge_datasets.ipynb
│   └── 06_analysis_and_plots.ipynb
│
├── data/
│   ├── raw/
│   │   ├── module_bank/            ← scraped from Exeter's public Module Bank
│   │   │   ├── index_2019-0.csv … index_2024-5.csv     ← per-year module lists
│   │   │   ├── index_all.csv                           ← combined index
│   │   │   ├── modules_details.csv                     ← every module descriptor parsed
│   │   │   └── stem_corrections.csv                    ← progress file from notebook 02.5
│   │   └── nss/                    ← NSS .csv exports downloaded from the Office for Students
│   │       └── NSS{23,24,25}_10007792_{r,t}.csv     ← _r = registered cut, _t = taught-at cut
│   │
│   └── processed2/                 ← outputs of notebooks 03–06
│       ├── modules_clean.csv               ← one row per (module × year), cleaned
│       ├── dept_year_summary.csv           ← one row per (department × year)
│       ├── nss_exeter_long.csv             ← NSS results in long form, registered cut
│       ├── nss_exeter_wide.csv             ← NSS results pivoted to one row per (subject × year)
│       ├── nss_exeter_taughtat_long.csv    ← NSS results, taught-at cut (robustness check)
│       ├── nss_exeter_taughtat_wide.csv
│       ├── dept_to_nss_subject.csv         ← Exeter dept code → CAH2 subject mapping
│       ├── merged_dept.csv                 ← one row per (dept × NSS year), used by the model
│       ├── merged_subject.csv              ← one row per (subject × NSS year), used by headline charts
│       └── blog_summary_table.csv          ← compact dept-by-subject table referenced in the post
│
├── output/figures/                 ← every PNG written by notebook 06
│   ├── chart_1_assessment_mix.png             ← blog chart 1
│   ├── chart_2_trends.png                     ← blog chart 2
│   ├── chart_3_feature_importance.png         ← blog chart 3
│   ├── chart_4_pred_vs_actual.png             ← blog chart 4
│   └── diag_a_…  diag_b_…  diag_c_…  diag_d_… ← diagnostic charts kept out of the post
│
└── _archive/                       ← items kept for reference but NOT used by the pipeline
    ├── README.md                              ← what's in here and why
    ├── docs/                                  ← course brief PDF
    ├── figures_old/                           ← chart PNGs from earlier project iterations
    ├── module_bank_old/                       ← pre-STEM-fix backup of the scrape
    └── nss_xlsx/                              ← original Excel downloads from the OfS
```

---

## Data

Two public sources, joined on subject.

| Source | What it is | Where it came from | Licence |
|---|---|---|---|
| **Module Bank** | One descriptor per (module × academic year) for 2019/20 through 2024/25, including assessment percentages, scheduled and independent study hours, anticipated cohort size, and the count of summative items. | Scraped from `https://hub.exeter.ac.uk/esami/modules/` | Public-facing institutional information; no login required. The scrape uses a one-second polite delay between requests. |
| **National Student Survey** | Subject-level positivity scores across the eight NSS themes for survey years 2023, 2024 and 2025. Filtered to UKPRN 10007792 (University of Exeter), full-time undergraduates, CAH2 subject level. | Downloaded from the Office for Students NSS data release pages. | Released for general use by the OfS. |

Pre-2023 NSS data is not used because the Office for Students rewrote the survey instrument in 2023 (new question wording, 4-point scale, new theme grouping) and itself flags pre and post 2023 results as not directly comparable.

### Data dictionary — the variables that feed the model

The eight features fed into the random forest live in `data/processed2/merged_dept.csv`:

| Variable | Type | Definition |
|---|---|---|
| `mean_coursework_pct` | float | Department-year mean of the coursework share of summative marks, credit-weighted across modules. |
| `mean_written_exam_pct` | float | Same as above for written exams. |
| `mean_practical_pct` | float | Same as above for practicals. |
| `mean_class_size` | float | Mean of the anticipated student count per module. |
| `mean_scheduled_hours` | float | Mean scheduled (i.e. timetabled) study hours per module. |
| `mean_contact_ratio` | float | Scheduled hours / total study hours, averaged across modules. |
| `mean_assess_diversity` | float | Mean count, per module, of how many of {coursework, written exam, practical} carry a non-zero share. Range 1–3. |
| `mean_n_summative_items` | float | Mean count of separately-marked pieces of assessment per module. |

Target: `nss_assessment_feedback`, the dept-year positivity score (0–100) on the NSS Assessment & Feedback theme.

---

## Reproducing from scratch

The cleaned and merged CSVs are already in `data/processed2/`, so notebooks 03–06 will run on a clean clone without re-scraping. If you want to rebuild the scrape as well:

1. **Run notebook 01.** Visits the Module Bank for each academic year (2019/20 – 2024/25) and lists every module offered. Writes one CSV per year plus the combined `index_all.csv`. ~1 minute per year.
2. **Run notebook 02.** For every (module, year) in the index, fetches the descriptor page, parses the assessment percentages, scheduled / independent hours, anticipated cohort size, summative item count, and writes everything to `modules_details.csv`. **Long step:** about 5 hours with the polite 1-second delay.
3. **Run notebook 02.5.** Re-fetches the ~1,549 STEM rows (MTH, ECM, ENG, PHY) where notebook 02 hit the wrong page layout and got blank fields back. **About 25–30 minutes.** Saves progress every 50 rows so it can resume after an interruption.
4. **Run notebook 03.** Cleans the scrape: pulls dept code, level and faculty out of the module code; computes total hours, contact ratio and assessment diversity; renormalises the assessment percentages to sum to 100; keeps only undergraduate rows; aggregates to one row per (department × year).
5. **Run notebook 04.** Reads the three NSS spreadsheets, filters to full-time UGs / CAH2 / Exeter, averages the 26 question rows up to the 8 NSS themes, writes a long and wide form for each of the registered (`_r`) and taught-at (`_t`) cuts, and builds the dept-code → CAH2 subject mapping.
6. **Run notebook 05.** Joins the dept-year module summary to the NSS theme scores. Produces `merged_dept.csv` (used by the model) and `merged_subject.csv` (used by the headline charts).
7. **Run notebook 06.** Builds the four blog charts and the diagnostic charts. Fits the random forest with 5-fold cross-validation, writes feature importances and predicted-vs-actual scores. Outputs go to `output/figures/`.

Don't lower the scraping sleep below 1 second. Exeter's web team is friendly and we should keep it that way.

---

## Methodology summary

- **Cleaning.** Renormalise rather than drop assessment-percentage rows whose three channels don't sum to 100 (covers ~1,600 module-years that an earlier strict version of the pipeline lost). Keep only undergraduate modules. Aggregate up using a credit-weighted mean.
- **Linking the two halves.** Modules are grouped by their three-letter code prefix (BEE for Economics, PYC and PSY for Psychology, etc.). A hand-built mapping table connects each prefix to the NSS CAH2 subject that surveys that cohort.
- **Model.** Random forest regressor (scikit-learn), 500 trees, minimum leaf size 2, target = NSS Assessment & Feedback positivity, 5-fold cross-validation. With 72 dept-year rows, the cross-validated R² is the only honest measure of fit.
- **Robustness.** A diagnostic chart re-fits the model separately for each of the eight NSS themes, and a second compares the registered-at and taught-at NSS cuts (which turn out to be identical at Exeter). Both are saved to `output/figures/` as `diag_*.png` and discussed only briefly in the post.

---

## Notes, quirks and known issues

A handful of things that surfaced during the project. Useful to keep in mind, and the most important caveats are also discussed in the blog's Assumptions and limitations section.

## Graphs


The diagnostics section in the graphs folder is the optional charts  (`diag_*.png`) that are useful but didn't make the blog cut.


### "Registered at" and "taught at" are the same population at Exeter

The OfS publishes NSS results in two cuts: `_r` (students *registered at* the provider, regardless of where taught) and `_t` (students *taught at* the provider, including those registered with partner providers). For institutions with big franchise / partner-college arrangements these two can diverge. For Exeter, they are bit-for-bit identical, so the registered/taught-at robustness check is a perfect 45° line. There are no partner-provider arrangements feeding the totals.This can be seen in graph: files\output\figures\chart_8_robustness_r_vs_t.png

### The renormaliser is intentionally lenient

Notebook 03 doesn't drop rows whose three assessment percentages don't sum to exactly 100, it rescales them as long as the sum is in the range 10–200. A strict version of the pipeline lost ~1,600 module-years to what was almost always a rounding issue or a single missing channel.

### The model has 72 rows because of two layered constraints

1. **NSS in this format only goes back to 2023** (see *Data* above). Years before 2022/3 in the module data have no NSS counterpart.
2. **Some Exeter departments have no UG modules** that map to a published NSS subject. Sport (SHS), Medicine (MED, MDC), Nursing (NUR) and most of the language codes either don't run separately-listed UG modules or only run them in years we couldn't match.
3. **Cross-faculty modules can't be mapped** to a single CAH2 subject. The 11 EMP (industrial experience) rows are the only ones flagged `Unknown`.

### Subject-level vs dept-level: two merged files, two uses

Some CAH2 subjects pool several Exeter departments (e.g. *Sociology, social policy and anthropology* = SOC + SPA + ANT; *History and archaeology* = HIH + ARC). Notebook 05 produces:

- `merged_subject.csv` — one row per (CAH2 subject × year). Used by the headline subject-level charts.
- `merged_dept.csv` — one row per (Exeter dept × year). Each dept gets the NSS scores of its parent CAH2 subject. Used by the model, which gains rows from this expansion (72 vs the ~25 you'd get if you stuck to subject level only).

`merged_dept.csv` has one quirk: ECM (Engineering and Computing) appears twice per year, once mapped to *Computing* and once to *Engineering*, giving it 6 rows in the model rather than 3. Identical predictors with different NSS targets. It's a 4-row-out-of-72 effect.

### Late fixes

Two issues turned up after the first model run and were fixed in place. Both are documented in the relevant notebook cells:

| Issue | What was wrong | Fix |
|---|---|---|
| **PSY missing from mapping** | Two psychology codes exist (`PSY`, the main UG dept; `PYC`, a small Stage-3 specialist programme). The first version mapped only `PYC`, so the bulk of UG psychology was being silently dropped. | Added `('PSY', 'Psychology')` alongside `PYC` in [notebook 04 §9](notebook2/04_process_nss3.ipynb). |
| **STEM modules missing assessment data** | MTH, ECM, ENG and PHY pages use a different template (`sys=1` instead of `sys=0`). Notebook 02 hit them with the wrong system flag and got empty pages back, dropping ~850 STEM modules silently. | New [notebook 02.5](notebook2/02_5_rescrape_stem_modules.ipynb) re-fetches the broken rows with `sys=1` and a parser that handles the new layout. |

After these two fixes the model went from 54 to 72 rows, and Psychology, Maths, Computing, Engineering and Physics all entered the analysis.

---

## Tools used

- **Python 3.10+** for everything.
- **Web scraping:** `requests` and `BeautifulSoup` against Exeter's public Module Bank.
- **Data wrangling:** `pandas`, `numpy`.
- **Modelling:** `scikit-learn` (random forest, K-fold CV) — this is the unit-5 element of the project.
- **Plots:** `matplotlib` and `seaborn`.
- **Notebooks:** Jupyter.
- **Front-end:** plain HTML and CSS for `blog.html`. No JavaScript framework, no build step.

See `requirements.txt` for the exact package list.

---

## Author

Final empirical project for BEE2041 (Data Science in Economics), University of Exeter, April 2026.

Code is provided for academic replication. Module Bank descriptors and NSS results remain the property of their original publishers (the University of Exeter and the Office for Students respectively); only aggregated figures are reproduced in the blog.
