# ThreatOrator–GraphAEGIS: re-implementation, predictions and metric recomputation

Companion archive for the *Scientific Reports* manuscript “Proactive Detection of Unseen Network Intrusion Attacks Using a
Hybrid BERT and Graph Neural Network Architecture” (revision 2).

**What this archive is.** All results in the revised manuscript were produced by the code in `src/`. It is a
re-implementation of the framework and of the six baselines, written for the revision. It is not the code of the first
version of the manuscript. The re-run used seeded, stratified 50,000-flow subsamples of UNSW-NB15 and NF-UNSW-NB15-v3, BERT-mini as the
language model (CPU only), and a corpus of CVE descriptions (CVE year ≤ 2014) from the National Vulnerability Database.
The manuscript (Methods, “Data sampling”) describes these choices.

## Contents

| Path | Content |
|---|---|
| `src/prepare_data.py` | Step 1: seeded subsample (seed 42), flow features, 60-s host windows, stratified 10 folds |
| `src/threatorator.py` | Step 2: CVE corpus, keyword family labels, ThreatOrator (BERT-mini + additive domain mask), MLM pre-training, fine-tuning, (port, protocol) retrieval index |
| `src/models.py` | GraphAEGIS, fusion model and the six baselines (DNN-IDS, CNN-IDS, GraphSAGE, GAT-IDS, LSTM, BS-GAT) |
| `src/run_cv.py` | Step 4: stratified 10-fold CV of all models; saves fold-wise predictions, learning curves, gate values, early-detection and noise results |
| `src/run_logo.py` | Step 5: leave-one-group-out (Worms, Backdoors) |
| `src/recompute_metrics.py` | Recomputes every metric of Tables 2–6 and Supplementary Table S2 from the saved predictions (per-fold confusion matrices, mean ± SD, 95% t-interval, paired t-test, check that accuracy lies between recall and specificity) |
| `src/aggregate.py`, `src/make_numbers.py`, `src/figures.py` | Collect the remaining results and draw the figures from the saved outputs |
| `src/export_indices.py` | Writes `data/subsample_indices/` and checks that the indices rebuild the subsamples exactly |
| `predictions/` | Saved test-set predictions of every model and fold (`<dataset>/<model>/fold<k>.csv`) and of the LOGO experiment (`logo/`) |
| `results/recomputed/` | Tables 2–6 and Supplementary Table S2 (fold-wise TP, TN, FP, FN and metrics) as produced by `recompute_metrics.py` |
| `results/numbers.json`, `results/summary.json` | Every number quoted in the text, with its source |
| `data/subsample_indices/` | Row positions, family and fold of the 50,000 sampled flows of each dataset |
| `data/sampling_info.json` | Flow counts per family in the full datasets and in the subsamples (Supplementary Table S1) |
| `data/cve_corpus_labels.csv` | CVE identifiers of the corpus with year, keyword-derived family label (empty = unlabelled) and extracted ports/protocols |

## Verify the reported metrics (no GPU, a few seconds)

```
cd src
python recompute_metrics.py --data ../predictions --out ../recomputed_check
```
The output is identical to `results/recomputed/`.

## Full re-run

1. Install the packages in `requirements.txt` (Python 3.13).
2. Put the datasets in `data/`:
   - UNSW-NB15: the four CSV files of the original release (https://research.unsw.edu.au/projects/unsw-nb15-dataset), concatenated in order 1–4, saved as `data/raw_alt.parquet`.
   - NF-UNSW-NB15-v3 (https://doi.org/10.48610/6E0EDA1), saved as `data/NF-UNSW-NB15-v3.parquet`.
   - The NVD CVE descriptions (columns `CVE-ID`, `DESCRIPTION`), saved as `data/cve.csv`.
   - The BERT-mini weights (`google/bert_uncased_L-4_H-256_A-4`), saved in `data/bert-mini/`.
3. Run in `src/`:
```
python prepare_data.py
python threatorator.py
python run_cv.py --ds UNSW-NB15
python run_cv.py --ds NF-UNSW-NB15-v3
python run_logo.py
python recompute_metrics.py --data ../predictions --out ../results/recomputed
python aggregate.py && python make_numbers.py && python figures.py
```
On a 4-core CPU one fold of all models takes about 7–9 minutes.

## Licence and citation

MIT (see `LICENSE`). Please cite the article (`CITATION.cff`). The datasets and the NVD data are subject to their own terms
and are not redistributed here.
