# Smartphone-Recorded Cough Sounds to Predict Chest X-Ray Abnormalities in Adults Evaluated for Lower Airway Disease

**R2D2 TB Network | Multi-country diagnostic accuracy study**

## Overview

Chest X-ray (CXR) is a WHO-recommended triage tool for tuberculosis (TB), but acquiring and interpreting radiographs requires infrastructure and trained readers that are scarce in high-burden primary care settings. This project investigates whether smartphone-recorded cough audio can serve as a pre-imaging triage signal — a lightweight, infrastructure-free approach to predicting CXR abnormality.

We compare three cough audio processing pipelines, with and without clinical/demographic features, across 1,058 adults undergoing TB evaluation in five countries.

---

## Study Population

Data come from the **Rapid Research in Diagnostic Development TB Network (R2D2)**, a prospective diagnostic accuracy study. Participants were adults presenting with ≥2 weeks of new or worsening cough at primary care sites in:

| Country | n |
|---|---|
| Uganda | 300 |
| Philippines | 248 |
| Vietnam | 241 |
| South Africa | 184 |
| India | 85 |
| **Total** | **1,058** |

Cough recordings (~10 seconds each) were collected via the **Swaasa smartphone application** at enrollment. CXR labels (normal/abnormal) were derived from QXR computer-aided diagnostics or radiologist annotation. The sample was balanced: 567 abnormal, 491 normal.

---

## Methods

### Audio Pipelines

Three preprocessing and feature extraction strategies were evaluated:

#### 1. Full Recording Pipeline — `feature_extraction_all_new.ipynb`
Features extracted from the entire preprocessed recording, including all cough bursts and inter-cough intervals. Preprocessing: resampled to 16 kHz, converted to mono, Butterworth bandpass filter (100–7,500 Hz). Features include MFCCs with first and second derivatives, mel spectrogram statistics, spectral features, temporal features, time-domain statistics, and STFT features, plus 10 recording-level temporal features (cough count, rate, duration statistics, inter-cough intervals, energy ratio, etc.).

#### 2. Segmented Cough Pipeline — `feature_extraction_segment_new.ipynb`
An energy-based algorithm isolates individual cough events (silence threshold: 21 dB; minimum segment: 0.32 s). Segment-level predictions are aggregated to subject level via weighted averaging proportional to segment duration. Adds voice quality measures (jitter, shimmer, HNR, formants F1–F4) via Parselmouth.

#### 3. HeAR Pipeline — `hear_model_full_new.ipynb`
Uses Google's **Health Acoustic Representations (HeAR)** model — a self-supervised model trained on 313 million two-second health audio clips — to generate 512-dimensional embeddings per segment without manual feature engineering. Embeddings are compressed via PCA (retaining 96% variance → 143 components) or a fully connected autoencoder (128-dimensional latent space), then aggregated using mean pooling.

### Modeling

All pipelines were split 80/20 at the subject level (fixed across pipelines). Feature selection used univariate logistic regression (p ≤ 0.40, applied only to acoustic pipelines). Features were scaled with RobustScaler. Three classifiers were tested per pipeline: **logistic regression**, **SVM**, and **random forest**.

Performance metrics: AUROC, sensitivity, specificity, accuracy. Given the TB triage context, sensitivity was treated as a co-primary criterion.

---

## Key Results

### Audio Only

| Pipeline | Best Classifier | AUROC | Sensitivity | Specificity |
|---|---|---|---|---|
| HeAR + PCA | Logistic Regression | **0.759** | 74.0% | 55.4% |
| Segmented Cough | Logistic Regression | 0.708 | 73.0% | 53.6% |
| Full Recording | Logistic Regression | 0.656 | — | — |

### Audio + Clinical/Demographic Features

| Pipeline | Best Classifier | AUROC | Sensitivity | Specificity |
|---|---|---|---|---|
| HeAR + PCA | Logistic Regression | **0.822** | 76.0% | 67.0% |
| Segmented Cough | Logistic Regression | 0.811 | 76.1% | 73.5% |
| Full Recording | Logistic Regression | 0.803 | — | — |

Adding 15 clinical/demographic variables (age, sex, BMI, HIV status, diabetes, symptoms, etc.) produced the largest performance gain across all pipelines, narrowing differences between acoustic approaches. All pipelines converged to AUROC 0.80–0.82 with demographics included.

> **Note:** These results fall below WHO Target Product Profile thresholds (≥90% sensitivity, ≥80% specificity) for a TB triage test. Substantial country-level and sex-based heterogeneity persists, highlighting the need for local calibration.

---

## Subgroup Findings

- **Country:** South Africa showed the strongest balanced performance (sensitivity 84.6%, specificity 80.0%, AUROC 0.865). Vietnam showed near-perfect sensitivity with very low specificity, suggesting systematic over-prediction of abnormality.
- **Age:** Sensitivity increased monotonically with age (57.1% in under-25s → 87.9% in 65+).
- **Sex:** Female participants showed higher sensitivity (82.8%) but lower specificity (41.5%) than males; this gap persisted after adding demographic features.

---

## Repository Contents

| File | Description |
|---|---|
| `feature_extraction_all_new.ipynb` | Full recording pipeline: preprocessing, feature extraction, and model training |
| `feature_extraction_segment_new.ipynb` | Segmented cough pipeline: energy-based segmentation, feature extraction, and modeling |
| `hear_model_full_new.ipynb` | HeAR embedding pipeline: segment extraction, PCA/autoencoder compression, and modeling |
| `CXR_Poster.pdf` | Conference poster |

---

## Dependencies

- `librosa`, `parselmouth`, `scikit-learn`, `numpy`, `pandas`
- HeAR model ([Google Health Acoustic Representations](https://github.com/Google-Health/google-health/tree/master/health_acoustic_representations))

---

## Citation

Manuscript in preparation. R2D2 TB Network, 2026.
