# Side-Channel Attacks Defeat Protection in 3D Printers

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)
[![Dataset: Zenodo](https://img.shields.io/badge/dataset-Zenodo%2013329934-blue.svg)](https://zenodo.org/records/13329934)

---

## Overview

This repository contains the full research pipeline for the paper **"Side-Channel Attacks Defeat Protection in 3D Printers"** — the first empirical evaluation of Active Motor Noise Cancellation (AMNC) as a hardware countermeasure against multi-modal side-channel attacks on FDM 3D printers.

**Key finding:** AMNC fully neutralizes acoustic side-channel leakage (9.79% accuracy, *p* = 0.305, indistinguishable from random chance) but provides **zero protection** against vibration-based attack (100% accuracy, 95% CI [97.38%, 100%], *p* < 0.0001). A single low-cost accelerometer defeats the countermeasure entirely.

---

## Results Summary

| Channel | Pre-Attack | Post-Attack | Post-Defense |
|---|---|---|---|
| Acoustic-Only | 9.79% | 9.79% | 11.19% |
| Vibration-Only | **100.00%** | **100.00%** | **100.00%** |
| Fused | **100.00%** | **100.00%** | **100.00%** |
| **Defense successful?** | — | No | No |

Vibration accuracy of 100% holds across **4 classifiers**, **3 individual accelerometer axes**, and **2 printers independently** (McNemar's test, *p* < 0.0001).

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
├── printer_sidechannel.tex              # IEEE Transactions LaTeX manuscript
├── references.bib                       # BibTeX references (35+ citations)
├── README.md                            # This file
└── figures/                             # Publication figures (PNG, 300 DPI)
    ├── fig1_pca_col1.png                # Pre-attack PCA discriminability
    ├── fig2_accuracy_col1.png           # Accuracy by condition
    ├── fig3_robustness_col1.png         # Robustness profile
    ├── fig4_confusion_col1.png          # Confusion matrix (fused, post-defense)
    ├── fig5_spectrogram_col1.png        # AMNC spectral effect
    ├── fig6_defense_col1.png            # Defense impact by channel
    ├── fig7_perobject_col1.png          # Per-object accuracy
    ├── fig8_ablation_classifiers_col1.png  # Ablation by classifier
    └── fig9_ablation_axes_col1.png      # Ablation by vibration axis
```

---

## Experimental Design

The study uses a four-phase evaluation framework:

1. **Pre-Attack Baseline** — PCA of raw feature spaces to establish signal discriminability before any classifier is trained
2. **Post-Attack** — Supervised classification under acoustic-only, vibration-only, and fused modalities
3. **Post-Defense** — Re-evaluation after simulated AMNC notch filter applied to acoustic channel
4. **Defense Evaluation** — Ablation across classifiers, axes, and printers with full statistical testing

---

## Methods

### Feature Extraction

| Channel | Features | Dimensionality |
|---|---|---|
| Acoustic | 13 MFCCs + spectral centroid/bandwidth/rolloff/ZCR (mean + std) | 32 |
| Vibration | Mean, std, RMS, peak-to-peak, dominant FFT freq × 3 axes | 15 |
| Fused | Acoustic + Vibration concatenated | 47 |

### AMNC Defense Simulation

AMNC is modeled as a notch filter bank targeting stepper motor resonances:
- **Center frequencies:** 120, 180, 240, 300, 360 Hz
- **Quality factor:** Q = 30
- **Implementation:** `scipy.signal.iirnotch` (second-order IIR)

### Classification

- **Primary:** Random Forest (200 estimators), StandardScaler pipeline
- **Ablation:** Gradient Boosting, SVM (RBF, C=10), KNN (k=5)
- **Validation:** Stratified 5-fold cross-validation

### Statistical Tests

- Wilson confidence intervals for proportions
- One-sided binomial test against 8.33% random baseline (1/12 classes)
- McNemar's test for paired classifier comparison

---

## Ablation Results

| Classifier | Acoustic (%) | Vibration (%) | Fused (%) |
|---|---|---|---|
| Random Forest | 9.79 | **100.00** | **100.00** |
| Gradient Boost | 11.19 | **100.00** | **100.00** |
| SVM (RBF) | 9.79 | **100.00** | 48.25 |
| KNN (k=5) | 6.29 | **100.00** | 12.59 |
| RF — X-axis only | — | **100.00** | — |
| RF — Y-axis only | — | **100.00** | — |
| RF — Z-axis only | — | **100.00** | — |
| RF — A1 Mini only | — | **100.00** | — |
| RF — P1P only | — | **100.00** | — |

---

## Setup and Reproduction

### Requirements

```bash
pip install librosa scikit-learn pandas numpy matplotlib seaborn scipy pydub statsmodels zenodo-get pymupdf
```

### Google Colab (Recommended)

1. Open `3D_Printer_SideChannel_Study.ipynb` in Google Colab
2. Mount your Google Drive
3. The notebook will automatically download the dataset from Zenodo to `MyDrive/3dprinter_sidechannel/`
4. All results and figures save to `MyDrive/3dprinter_sidechannel/results/`
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

