# The Arousal Index

**An interpretable, calibrated, cross-dataset model of musical arousal.**

This repository accompanies the paper *"[INSERT PAPER TITLE]"* ([INSERT arXiv LINK once posted]). It contains a compact, fully interpretable model that predicts the **arousal** (felt activation/energy) a listener experiences from a piece of music, on a calibrated 0–11 scale, from six named acoustic features. The headline result: a **decorrelated six-feature model transfers across datasets substantially better (Spearman 0.77) than a twelve-feature model using the same underlying features (Spearman 0.60)** — evidence that resolving feature collinearity improves both interpretability *and* cross-corpus generalization.

> **A note on terminology.** This repository is the umbrella for a research program targeting *musical intensity*. Its first model (this work) honestly measures **arousal** as operationalized by human listener ratings — a related but distinct construct that includes production factors such as loudness and mastering. Work on *intensity* proper (which isolates sonic aggression from production) is future work. Throughout, "arousal" means what the model measures.

## Headline results

| Model | Within-DEAM R² (5-fold) | Transfer to PMEmo (Spearman) |
|---|---|---|
| **Decorrelated 6-feature (ours)** | 0.29 | **0.77** |
| Full 12-feature (reference) | 0.42 | 0.60 |
| Black-box, 260 features (ceiling) | 0.57 | — |

The interpretable model is *less* accurate within a single dataset but generalizes *far better* to an unseen one — the twelve-feature model's higher within-dataset R² is partly memorization of dataset-specific feature collinearity, which the transfer test exposes.

## What's here

```
├── arousal-index.ipynb          # the complete pipeline, runs top-to-bottom
├── deam_librosa_features.csv     # pre-extracted features for DEAM (1802 songs)
├── pmemo_librosa_features.csv    # pre-extracted features for PMEmo (767 songs)
├── arousal_index_curves.png      # Figure: interpretable feature-effect curves
├── requirements.lock             # pinned dependency versions
└── README.md
```

## Reproducing the results

### 1. Environment
```bash
pip install -r requirements.lock
```
(Python 3.11+. The pinned versions are those used to produce the reported numbers.)

### 2. Data
This repo ships the **pre-extracted features** (the two `.csv` files), so you can reproduce every reported number **without downloading any audio**. Just run the notebook — it detects the CSVs and uses them (the "fast path").

To re-extract features from scratch, or to use the audio-based scoring tool, download the two public source datasets (both free for non-commercial research; not redistributed here due to size and licensing):

- **DEAM** (MediaEval Emotion in Music): https://www.kaggle.com/datasets/imsparsh/deam-mediaeval-dataset-emotional-analysis-in-music
- **PMEmo** (2018): https://www.kaggle.com/datasets/adityaraghuvanshi999/pmemo-original

The feature extraction recipe is defined in `extract_six()` in the notebook (cell 2) — it is the single source of truth for how the CSVs were produced.

### 3. Run
Open `arousal-index.ipynb` and run all cells top to bottom. On the fast path (using the shipped CSVs) it completes in well under a minute and reproduces: within-dataset R² = 0.29, cross-dataset transfer Spearman = 0.77, the anchor ladder, and the feature-effect figure.

You can also run it in the cloud: https://www.kaggle.com/code/connerofarrell/arousal-index.

### Scoring your own audio
After running the notebook, the `score_arousal()` function scores any audio file 0–11:
```python
score_arousal('your_song.mp3')   # -> e.g. 6.5
```
Note: the model is trained on DEAM (predominantly folk/acoustic/light-rock) and will **underestimate genuinely extreme music** (e.g., metal), which lies outside its training range — a known limitation discussed in the paper and motivating future work with wider-range stimuli.

## Method in brief
Six interpretable acoustic features — loudness (RMS), spectral brightness (centroid), attack density (onset strength / spectral flux), noisiness (zero-crossing rate), a spectral-flatness sharpness proxy, and a flatness-derived harmonicity proxy — each summarized by its mean and standard deviation across 30-second clips. Because several of these are highly collinear (brightness ≈ noisiness ≈ sharpness, r > 0.9), the interpretable model retains one representative per correlated cluster (zero-crossing rate, loudness, harmonicity). A standardized Ridge regression maps these to a 0–11 arousal scale, calibrated with isotonic regression. Evaluated within-dataset by 5-fold cross-validation and across datasets by training on DEAM and testing, unchanged, on PMEmo (features extracted identically on both).

## Data & licensing
DEAM and PMEmo are the property of their respective creators and are used here for non-commercial research under their stated licenses; see the linked dataset pages. The pre-extracted feature CSVs in this repository are derived summary statistics released to support reproducibility.

**Please cite the original datasets** in any derivative work:
- DEAM: Aljanaki, A., Yang, Y.-H., & Soleymani, M. (2017). *Developing a benchmark for emotional analysis of music.* PLOS ONE. [verify full citation]
- PMEmo: Zhang, K., Zhang, H., Li, S., Yang, C., & Sun, L. (2018). *The PMEmo dataset for music emotion recognition.* ICMR. [verify full citation]

## Citation
If you use this work, please cite:
```
[INSERT BibTeX once the paper is finalized/posted]
```

## License
Code in this repository: MIT License. Derived feature data: released for non-commercial research consistent with the source datasets' licenses.
