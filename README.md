# Side-Channel Attacks Bypass Protection in 3D Printers

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)
[![Dataset: Zenodo](https://img.shields.io/badge/dataset-Zenodo%2013329934-blue.svg)](https://zenodo.org/records/13329934)

---

## Overview

This repository contains the full research pipeline for the paper **"Side-Channel Attacks Bypass Protection in 3D Printers"** — the first empirical evaluation of Active Motor Noise Cancellation (AMNC) as a hardware countermeasure against acoustic and vibration side-channel analysis on FDM 3D printers.

**Key finding:** AMNC neutralizes the acoustic side-channel (12.50% accuracy, at the 8.33% random baseline) but does not cover the vibration channel. Vibration still classifies the printed object: **~31% with summary statistics** and **~61% with a full-sequence temporal model**. The signal is amplitude-driven, device-specific, and does not support full geometric reconstruction.

---

## Results Summary

| Method | Accuracy | Note |
|---|---|---|
| Chance (1/12) | 8.33% | baseline |
| Acoustic (AMNC active) | 12.50% | ≈ chance — AMNC neutralizes acoustic |
| Vibration — summary, pooled | 31.25% | permutation *p* = 0.005 |
| Vibration — within A1 Mini | 36.11% | device-specific |
| Vibration — within P1P | 47.22% | device-specific |
| Vibration — full-sequence temporal (ordered) | **60.65%** | strongest result |
| Temporal — order-shuffled control | 32.64% | confirms sequential structure |

The ~28-point gap between the ordered (60.65%) and order-shuffled (32.64%) temporal models confirms a genuine sequential component tied to print progression.

---

## What the Vibration Channel Carries

Feature ablation localizes the signal to vibration **amplitude**, not waveform shape:

| Feature subset | Accuracy |
|---|---|
| All 15 summary features | 31.25% |
| Magnitude only (12) | 29.17% |
| Frequency only (3) | 6.25% |
| std + rms only (6) | 27.08% |

Top features (RF, mean decrease in impurity): `Z_mean`, `Z_rms`, `Y_p2p`, `Y_std`, `Z_p2p` — all amplitude statistics.

---

## Dataset

We use the **Madamopoulos–Tsoutsos 2024 dataset** (open access):

> Madamopoulos, C. & Tsoutsos, N.G. (2024). *3D Printer Audio and Vibration Side Channel Dataset*. Zenodo. https://doi.org/10.5281/zenodo.13329934

- **Printers:** Bambu Lab P1P (core-XY) and A1 Mini (bed-slinger) — both AMNC-equipped
- **Objects:** 12 geometrically diverse models (straight, circular, arced toolpaths)
- **Recordings:** 144 synchronized audio + accelerometer pairs (72 per printer)
- **Format:** `.mp3` / `.caf` audio, `.csv` accelerometer (X, Y, Z axes), `.gcode` ground truth

---

## Repository Structure

```
├── 3D_Printer_SideChannel_Study.ipynb   # Main Google Colab notebook
└── figures/                             # Publication figures (PNG, 300 DPI)
    ├── fig1_accuracy.png                # Accuracy by method
    ├── fig2_amplitude_vs_shape.png      # Amplitude vs frequency ablation
    ├── fig3_temporal_order.png          # Ordered vs order-shuffled temporal
    ├── fig4_feature_importance.png      # Top vibration features
    ├── fig5_confusion.png               # Confusion matrix (pooled vibration)
    └── fig6_crossprinter.png            # Cross-printer transfer
```

---

## Experimental Design

The pipeline runs five analyses:

1. **Acoustic under AMNC** — classification on acoustic features under active AMNC, plus a simulated notch filter and a quality-factor sweep
2. **Vibration — summary features** — pooled and within-printer object classification
3. **Feature ablation** — isolating amplitude from frequency content
4. **Full-sequence temporal model** — ordered vs order-shuffled accuracy
5. **Cross-printer transfer** — train on one printer, test on the other

---

## Methods

### Recording Pairing

Each recording session stores its audio alongside its own accelerometer capture. Every audio file is paired with the `.csv` capture in its **own session folder**, yielding 144 distinct audio–vibration pairs (6 per object per printer). Each recording is treated as a single labeled sample, and all samples derived from a recording are grouped together so that no recording is split across cross-validation folds.

### Feature Extraction

| Channel | Features | Dimensionality |
|---|---|---|
| Acoustic | 13 MFCCs + spectral centroid/bandwidth/rolloff/ZCR (mean + std) | 32 |
| Vibration (summary) | mean, std, RMS, peak-to-peak, dominant FFT freq × 3 axes | 15 |
| Vibration (temporal) | per-segment std/RMS/p2p/dominant-freq × 3 axes, 120 ordered segments | 120 × 12 |

### AMNC Defense Simulation

AMNC is modeled as a notch filter bank targeting stepper motor resonances:
- **Center frequencies:** 120, 180, 240, 300, 360 Hz
- **Quality factor:** Q = 30 (sensitivity checked at Q ∈ {15, 30, 60})
- **Implementation:** `scipy.signal.iirnotch` (second-order IIR)

AMNC filter sensitivity (acoustic accuracy): Q15 = 13.19%, Q30 = 9.72%, Q60 = 10.42% — near chance across all settings.

### Classification

- **Summary features:** Random Forest (200 estimators), StandardScaler pipeline
- **Temporal:** dilated 1D-CNN over the full 120-step sequence, averaged over 3 seeds
- **Ablation:** Gradient Boosting, SVM (RBF, C=10), KNN (k=5)
- **Validation:** grouped 5-fold cross-validation (one sample per recording)

### Statistical Tests

- Wilson confidence intervals for proportions
- One-sided binomial test against the 8.33% random baseline (1/12 classes)
- Label-permutation null (200 permutations)
- Order-shuffle control for the temporal model

---

## Ablation Results

Classifier ablation on vibration summary features (pooled):

| Classifier | Accuracy (%) |
|---|---|
| Random Forest | 31.25 |
| Gradient Boosting | 21.53 |
| SVM (RBF) | 9.03 |
| KNN (k=5) | 11.11 |

Cross-printer transfer (vibration): A1 Mini → P1P = 11.11%, P1P → A1 Mini = 12.50% — near chance, confirming device specificity.

---

## Applications & Security Implications

- **AMNC is an acoustic-only control.** It neutralizes the channel it targets but leaves structural vibration untouched. Mitigating the vibration channel requires dedicated measures — chassis-level isolation (damping feet, anti-vibration mounts), randomized stepper-acceleration profiles that de-correlate motion from toolpath geometry, or active vibration masking — each trading against print quality, speed, or hardware cost.
- **Device specificity is a defensive lever.** Vibration signatures do not transfer across printer architectures (cross-printer accuracy 11–12.5%), so a heterogeneous printer fleet imposes a per-device calibration cost on an attacker and provides partial natural protection that a homogeneous fleet does not.
- **Bounded threat.** The demonstrated capability is closed-set *identification* — confirming which of a set of known designs is on the bed — not reconstruction of unknown geometry. This supports monitoring or espionage against a known design catalog. The temporal result shows geometry-correlated information survives into the vibration channel, marking reconstruction (e.g., with the magnetic/power channels AMNC does not suppress) as the natural escalation.

---

## Setup and Reproduction

### Requirements

```bash
pip install librosa scikit-learn pandas numpy matplotlib seaborn scipy pydub statsmodels zenodo-get pymupdf torch
```

### Google Colab (Recommended)

1. Open `3D_Printer_SideChannel_Study.ipynb` in Google Colab
2. Mount your Google Drive
3. The notebook downloads the dataset from Zenodo to `MyDrive/3dprinter_sidechannel/`
4. Results and figures save to `MyDrive/3dprinter_sidechannel/results_v2/`
5. Run all cells in order

### Local Execution

```bash
git clone https://github.com/<your-username>/3dprinter-sidechannel
cd 3dprinter-sidechannel
pip install -r requirements.txt
# Download dataset manually from https://zenodo.org/records/13329934
# Update DRIVE_BASE path in notebook cell 2
jupyter notebook 3D_Printer_SideChannel_Study.ipynb
```

---

## Citation

Cite the dataset:

```bibtex
@misc{madamopoulos2024_zenodo,
  author    = {Madamopoulos, Christos and Tsoutsos, Nektarios Georgios},
  title     = {{3D} Printer Audio and Vibration Side Channels},
  year      = {2024},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.13329934}
}
```

---

## License

The dataset is subject to its own CC BY 4.0 license at Zenodo.
