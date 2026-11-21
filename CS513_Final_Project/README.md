# CS 513 Final Project: Predicting Unsafe NYC Building Facades

Stevens Institute of Technology, CS 513 (Data Mining & Machine Learning), Fall 2026.

**Question.** Can building traits and construction-safety history predict whether a
New York City facade inspection (FISP / Local Law 11) comes back **Unsafe**?

**Status: starter code. Nothing here has run on real data yet.** See
"What is verified and what is not" before trusting anything.

## Start here (first day)

```bash
pip install -r requirements.txt

python -m src.download_data      # 1. download the three raw files (large, be patient)
python -m src.data_check         # 2. the go / no-go numbers -> results/data_check_report.txt
python -m src.build_table        # 3. join everything -> data/processed/facade_table.csv
jupyter notebook notebooks/      # 4. one notebook per model
```

Step 2 will most likely tell you that some column names in `config.py` are wrong.
That is expected. Fix them in `FACADE_COLS` (and nowhere else) and run it again.

Post `results/data_check_report.txt` in the group chat. It holds the numbers for
the go / no-go decision:

1. How many initial filings are rated Safe, SWARMP and Unsafe.
2. What share of buildings match a PLUTO record (target: 90% or more).
3. What share of filings have a readable date.

## Who does what

| Teammate | Model notebook | Shared job |
| --- | --- | --- |
| 1 | `01_naive_bayes.ipynb` | Domain lead: problem statement, feature review |
| 2 | `02_knn.ipynb` | Data owner: `download_data`, `data_check`, `build_table` |
| 3 | `03_decision_tree.ipynb` | Deck owner: slides, references |
| 4 | `04_neural_network.ipynb`, `05_comparison.ipynb` | Project lead: repo, helpers, comparison, extras |

## Layout

```
config.py              settings everyone shares: seed, paths, dataset IDs, column names
src/download_data.py   downloads the raw files from NYC Open Data
src/data_check.py      the go / no-go check
src/build_table.py     joins filings + PLUTO + violations into one table
src/prep.py            the split by building and the data preparation
src/evaluate.py        tuning, scoring, baselines, the comparison table
src/plots.py           one chart style for all figures
notebooks/             00_final_report is the one notebook to hand in (problem,
                       data, exploration, results, conclusion); 01-04 are one
                       per model; 05 is the comparison
results/               comparison_table.csv and the data check report
docs/AI_PROMPT_LOG.md  required by the course AI policy
tests/make_fake_data.py  fake raw files for testing the code
```

## Rules built into the code

- **Split by building.** 80% of buildings train, 20% test, seed 513. No building is
  in both. Tuning uses 5-fold cross-validation grouped by building.
- **Initial filings only.** Amended and subsequent filings reveal the answer.
- **No future information.** Violations count only if issued before the filing
  date. A PLUTO alteration counts only if it happened before the filing year.
- **Three feature steps.** Every model is trained on filing features only (`f_`),
  then with PLUTO (`p_`), then with violation history (`v_`). The change between
  steps answers our question.
- **Same metrics for everyone.** ROC AUC, recall, precision and F1 on the Unsafe
  class, plus the confusion matrix. Accuracy is reported but is not the headline.
- **Label.** `is_unsafe` = 1 for Unsafe, 0 for Safe or SWARMP. The three-way rating
  is kept in the `rating` column for a possible second experiment.

One change from the written plan: there is no "open violations" feature. The
open or closed status in the violations file is today's status, which nobody
knew on the inspection date, so using it would leak the future.

## What is verified and what is not

| Item | Status |
| --- | --- |
| The code runs end to end | Tested on **fake** data only |
| Facade columns `bin`, `cycle`, `filing_type`, `current_status` | Seen on the dataset page |
| Other facade column names in `config.py` | **Guesses.** `data_check` reports which are wrong |
| PLUTO column names | Not verified |
| ECB columns `bin`, `issue_date`, `severity`, `violation_type` | Seen in a published schema |
| Any score in this repo | None yet. Fake-data scores mean nothing |

## Test the code without the real data

```bash
python -m tests.make_fake_data data_fake
CS513_DATA_DIR=data_fake CS513_RESULTS_DIR=data_fake/results python -m src.data_check
CS513_DATA_DIR=data_fake CS513_RESULTS_DIR=data_fake/results python -m src.build_table
```

Never put fake-data results in the slides.

## Course requirements

- Every code file starts with the course header (course, first name, last name,
  ID, purpose). Fill in the `<fill in>` lines.
- AI-generated code is marked with an `AI NOTICE` line, and the prompts are in
  `docs/AI_PROMPT_LOG.md`. Add an entry whenever AI writes more code.

## Sources

- DOB NOW: Safety, Facades Compliance Filings, NYC Open Data, dataset `xubg-57si`.
- Primary Land Use Tax Lot Output (PLUTO), NYC Dept. of City Planning, dataset `64uk-42ks`.
- DOB ECB Violations, NYC Open Data, dataset `6bgk-3dad`.
- Published filings-only model used as a reference point (ROC AUC 0.766):
  https://huggingface.co/iaac-maai/day2-team1-unsafe-facade
