# The Arousal Index

**An interpretable, calibrated, cross-dataset model of musical arousal.**

This repository accompanies a paper in preparation for ICASSP 2027 (draft complete; preprint link forthcoming). It contains a compact, fully interpretable model that predicts the **arousal** (felt activation/energy) a listener experiences from a piece of music, on a calibrated 0–11 scale, from **three named acoustic features — loudness, noisiness, and a tonality index — each summarized by its mean and variability (six descriptors)**, selected from six extracted features by a collinearity analysis. The headline result: this **decorrelated three-feature model transfers across datasets substantially better (Spearman 0.77) than a twelve-descriptor model built from the same six underlying features (0.60)** — and spline and gradient-boosted variants of the same six descriptors fit better in-corpus while transferring worse still (0.49, 0.40). A **6,373-functional openSMILE black-box** (same boosted learner, features extracted identically on both corpora) pushes within-corpus R² to **0.63** — the best of any model tested — while transferring at **0.43**: the largest capacity, the largest accuracy–transfer gap (bootstrap Δρ vs. linear +0.34, 95% CI [+0.28, +0.40]). In the reverse direction (train PMEmo → test DEAM) capacity again never helps: spline ties linear (ρ ≈ 0.59) and boosting drops to 0.52. Across both directions, added capacity never improved transfer — and severely harmed it when training on the broader corpus. Resolving collinearity and restraining capacity improved both interpretability *and* cross-corpus generalization.

> **A note on terminology.** This repository is the umbrella for a research program targeting *musical intensity*. Its first model (this work) honestly measures **arousal** as operationalized by human listener ratings — a related but distinct construct that includes production factors such as loudness and mastering. Work on *intensity* proper (which isolates sonic aggression from production) is future work. Throughout, "arousal" means what the model measures. A second terminology note used throughout the code and paper: a **feature** is a named frame-level acoustic quantity; a **descriptor** is its per-clip mean or standard deviation.

## Headline results

| Model | Within-DEAM R² (5-fold) | Transfer to PMEmo (Spearman) |
|---|---|---|
| **Decorrelated linear, 3 features / 6 descriptors (ours)** | 0.29 | **0.77** |
| Gradient-boosted (same 6 descriptors) | 0.37 | 0.40 |
| Spline / GAM (same 6 descriptors) | 0.39 | 0.49 |
| Full linear, 12 descriptors | 0.42 | 0.60 |
| Black-box (openSMILE ComParE 2016, 6,373 functionals) | 0.63 | 0.43 |

The interpretable model is the *least* accurate within a single dataset and generalizes *far better* than every higher-capacity alternative — added capacity partly memorizes dataset-specific structure, which the transfer test exposes. Selecting a model by within-dataset accuracy would pick the black-box and forfeit 0.34 of transfer correlation (0.77 → 0.43). Reverse direction (train PMEmo → test DEAM): linear ρ = 0.59, spline 0.59, boosted 0.52 — capacity never helps in either direction; the severe penalty is specific to training on the heterogeneous corpus.

## What's here

```
├── arousal-index.ipynb          # the complete pipeline, runs top-to-bottom
├── deam_librosa_features.csv    # pre-extracted librosa features, DEAM (1802 songs)
├── pmemo_librosa_features.csv   # pre-extracted librosa features, PMEmo (767 songs)
├── arousal_index_curves.png     # Figure: interpretable feature-effect curves
├── refs.bib                     # verified bibliography for the paper
├── requirements.lock            # pinned dependency versions
└── README.md
```

The two openSMILE feature tables used by the black-box reference (`deam_opensmile.csv`, `pmemo_opensmile.csv`, ~300 MB combined) exceed GitHub's file-size limits and are published separately as a Kaggle dataset: https://www.kaggle.com/datasets/connerofarrell/deam-and-pmemo-opensmile. The final notebook cell finds them automatically when that dataset is attached (or the CSVs are placed in the working directory), and otherwise re-extracts them from the source audio (~2.5 h; requires internet for `pip install opensmile`).

## Reproducing the results

### 1. Environment
```bash
pip install -r requirements.lock
```
(Python 3.11+. The pinned versions are those used to produce the reported numbers. `opensmile`, used only by the final black-box cell, is installed at runtime by that cell.)

### 2. Data
This repo ships the **pre-extracted librosa features** (the two smaller `.csv` files), so you can reproduce every core number **without downloading any audio**. Just run the notebook — it detects the CSVs and uses them (the "fast path").

To re-extract features from scratch, or to use the audio-based scoring tool, download the two public source datasets (both free for non-commercial research; not redistributed here due to size and licensing):

- **DEAM** (MediaEval Emotion in Music): https://www.kaggle.com/datasets/imsparsh/deam-mediaeval-dataset-emotional-analysis-in-music
- **PMEmo** (2018): https://www.kaggle.com/datasets/adityaraghuvanshi999/pmemo-original

The feature extraction recipe is defined in `extract_six()` near the top of the notebook — it is the single source of truth for how the librosa CSVs were produced. The openSMILE extraction recipe lives in the final cell and uses the identical 30 s / 22.05 kHz front end.

### 3. Run
Open `arousal-index.ipynb` and run all cells top to bottom. On the fast path, everything through the ablation study completes in well under a minute and reproduces: within-dataset R² = 0.29, cross-dataset transfer Spearman = 0.77, the reverse-direction results, the bootstrap confidence intervals, the anchor ladder, and the feature-effect figure. The final cell (the black-box reference) additionally reproduces R² = 0.63 / ρ = 0.43 — fast with the openSMILE CSVs available, ~2.5 h if re-extracting.

### Scoring your own audio
After running the notebook, the `score_arousal()` function scores any audio file 0–11:
```python
score_arousal('your_song.mp3')   # -> e.g. 6.5
```
Note: the model is trained on DEAM (predominantly folk/acoustic/light-rock) and will **underestimate genuinely extreme music** (e.g., metal), which lies outside its training range — a known limitation discussed in the paper and motivating future work with wider-range stimuli.

## Method in brief
Six interpretable acoustic features — loudness (RMS), spectral brightness (centroid), attack density (onset strength / spectral flux), noisiness (zero-crossing rate), spectral flatness, and a tonality index (negative log-flatness; monotone-related to flatness at the frame level) — each summarized by its mean and standard deviation across 30-second clips (twelve descriptors). Brightness, ZCR, flatness, and tonality form one highly inter-correlated spectral group (|r| = 0.62–0.86 on both corpora), so the interpretable model keeps the group's two least-correlated members (ZCR and tonality) plus loudness — three features, six descriptors. An ablation adding onset strength raised within-corpus fit (R² 0.29 → 0.35) but lowered transfer (ρ 0.77 → 0.65) — its correlation with arousal itself reverses sign between corpora (+0.40 DEAM vs −0.23 PMEmo) — so it is excluded. A standardized Ridge regression maps the six descriptors to a 0–11 arousal scale, calibrated with isotonic regression **fitted on out-of-fold predictions**. Evaluated within-dataset by 5-fold cross-validation and across datasets by training on DEAM and testing, unchanged, on PMEmo (features extracted identically on both), plus the reverse direction as a robustness check. The black-box reference applies the same gradient-boosted learner (200 trees, depth 3) to the 6,373 openSMILE ComParE 2016 functionals, extracted with the same front end on both corpora.

## Data & licensing
DEAM and PMEmo are the property of their respective creators and are used here for non-commercial research under their stated licenses; see the linked dataset pages. The pre-extracted feature tables in and linked from this repository (librosa and openSMILE) are derived summary statistics released to support reproducibility.

**Please cite the original datasets** in any derivative work:
- DEAM: Aljanaki, A., Yang, Y.-H., & Soleymani, M. (2017). *Developing a benchmark for emotional analysis of music.* PLOS ONE.
- PMEmo: Zhang, K., Zhang, H., Li, S., Yang, C., & Sun, L. (2018). *The PMEmo dataset for music emotion recognition.* ICMR.

## Citation
Citation details (BibTeX) will be added when the preprint is posted.

## License
Code in this repository: MIT (see [LICENSE](LICENSE)). Derived feature data: released for non-commercial research consistent with the source datasets' licenses.
