# Titanic Survival: an Integrated, Reproducible Data-Science Project

An end-to-end project that goes through the full data-science pipeline on the public Titanic passenger manifest:
**data collection and cleaning, exploratory analysis, statistical modelling, machine learning and visualization**, with one command that rebuilds every number and figure.

**Real-world problem.** In a mass-casualty maritime evacuation, which passenger characteristics were associated with survival, and how well can survival be predicted from information known at boarding? The answers show how access to safety was unequal and how far a data-driven model can go.

## Results at a glance

| Question | Answer |
|---|---|
| What mattered most? | **Sex, then class, and the women's advantage depended on class**: women outlived men by 60, 76 and 36 percentage points in first, second and third class (interaction p < 0.001). |
| Age? | Highest survival for children under 5 (68%); among males survival fell from 65% under 5 to 17% at 13 and over. The effect is non-linear (log form). |
| Fare, cabin, port? | Largely reflect class and who boarded where. Port has no significant independent effect (p = 0.19). |
| How well can we predict survival? | Test AUC **0.85** (95% interval 0.78 to 0.91) and 80% accuracy on 179 unseen passengers, against 61.5% for always predicting "died". |
| Is machine learning better than a classical model? | **Not demonstrably.** Four tuned models and a sex-by-class logistic regression score 0.865 to 0.889 AUC; differences are smaller than fold-to-fold variation. |
| Is it reproducible? | The integrated pipeline reproduces **14 of 14** headline numbers from the weekly deliverables, and the notebook and command-line runs give identical results. |

## Quick start

```bash
pip install -r requirements.txt
python run_pipeline.py          # full run, about 3 minutes on one CPU: data, results, model, figures
pytest -q                       # 13 tests (about 40 seconds)
jupyter nbconvert --to notebook --execute --inplace notebooks/00_integrated_project.ipynb   # master notebook
```

`python run_pipeline.py --fast` runs a reduced version for smoke tests; `--figures-only` rebuilds figures from saved results.

## Repository structure

```
titanic-ds-project/
├── README.md                    this file
├── requirements.txt             dependencies (tested with Python 3.12)
├── Makefile                     make pipeline | fast | test | notebook
├── run_pipeline.py              one command: ingest -> clean -> EDA -> inference -> ML -> figures
├── pytest.ini
├── src/titanic_project/         the codebase (one module per stage)
│   ├── config.py                paths, seeds, colour palette
│   ├── data.py                  Stage 1: loading, provenance (SHA-256), raw schema
│   ├── cleaning.py              Stage 2: cleaning with logged decisions, validation, data dictionary
│   ├── eda.py                   Stage 3: descriptive statistics, group rates, association tests, correlations
│   ├── inference.py             Stage 4: logistic regression, likelihood-ratio tests, assumption checks
│   ├── modeling.py              Stage 5: leakage-free pipelines, nested CV, threshold, test evaluation
│   ├── viz.py                   Stage 6: figures
│   ├── integrate.py             Stage 7: triangulation of findings across approaches
│   ├── results.py               result assembly, cross-check against weekly deliverables
│   └── utils.py                 Wilson interval, JSON helpers
├── tests/test_pipeline.py       unit and integration tests (including no-leakage tests)
├── notebooks/
│   ├── 00_integrated_project.ipynb   master notebook (runs everything, with commentary)
│   └── build_master_notebook.py
├── data/
│   ├── raw/titanic.csv          input, never modified
│   └── processed/               titanic_cleaned.csv (final dataset), data_dictionary.csv, split_ids.csv
├── results/                     results.json (every number in the report), test_predictions.csv, logs
├── models/final_model.joblib    fitted random-forest pipeline
├── figures/                     nine report figures (PNG, 200 dpi)
├── reports/                     integrated project report (Word and PDF)
└── weekly_deliverables/         Weeks 1-5 reports and notebooks, as submitted
```

## Pipeline stages and design decisions

1. **Ingest** (`data.py`): the raw file is loaded unmodified; its SHA-256 hash, shape, types and missingness are recorded.
2. **Clean** (`cleaning.py`): age imputed by median of title x class (with an `Age_Imputed` flag), port filled with the mode, cabin replaced by a flag and deck, fares flagged/capped/log-transformed, titles standardised. Nine automated checks; no rows removed.
3. **Explore** (`eda.py`): descriptive statistics, survival rates with 95% Wilson intervals, chi-square / Mann-Whitney / Kruskal-Wallis tests with Bonferroni correction and effect sizes, Spearman correlations.
4. **Statistical modelling** (`inference.py`): logistic regression with assumption checks (events per variable, VIF, Box-Tidwell), functional-form comparison for age, nested models compared by likelihood-ratio tests, odds ratios with confidence intervals, Hosmer-Lemeshow test, repeated cross-validation.
5. **Machine learning** (`modeling.py`): features rebuilt from the raw file; **all fitted preprocessing lives inside scikit-learn pipelines** and is re-fitted on every training fold; seven algorithms compared; **nested cross-validation** for tuning; a model-selection rule declared in advance; decision threshold chosen on training data only; **test set used once**.
6. **Visualization** (`viz.py`): nine figures with one colour language (blue = survived, vermillion = died; colour-blind-safe), data-driven titles, intervals and sample sizes.
7. **Integrate** (`integrate.py`, `results.py`): each finding is checked against all three analytical approaches, and headline numbers are cross-checked against the weekly deliverables.

## Reproducibility

* A single master seed (`config.RANDOM_STATE = 42`) controls the train/test split, models, searches and bootstraps.
* `results/results.json` holds every number used in the report; the notebook writes `results_notebook.json` and asserts that it is identical to it (wall-clock time excluded).
* Tests cover data invariants, determinism, independence of cleaning from the outcome, and absence of leakage (the age imputer learns only from training rows; predictions do not depend on other rows in the batch).
* Software versions of the reference run are stored in `results/results.json` under `environment`.

## Data and ethics

The data is the public Titanic manifest (source URL in `src/titanic_project/config.py`; file hash in `results.json`). It is observational and historical, so results are associations, not causal effects. The findings describe unequal access to safety in a historical disaster; they must **not** be used to rank people by sex or class in any real evacuation.

## Limitations

Several groups are small (for example 94 first-class women, 40 children under 5); 177 ages are imputed; passengers are not independent (families); and a 179-passenger test set gives wide intervals (AUC +/- 0.06).

## Rebuilding the report

`reports/Titanic_Integrated_Project_Report.docx` is generated from `results/results.json` and `figures/` by `reports/build/report6.js` (Node.js with the `docx` package: `cd reports/build && node report6.js`), so no number in the report is typed by hand.
