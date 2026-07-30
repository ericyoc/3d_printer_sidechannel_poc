# Side-Channel Attacks Survive Noise Cancellation in 3D Printers

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ericyoc/3d_printer_sidechannel_poc/blob/main/3d_printer_sidechannel_poc.ipynb)
[![Dataset: Zenodo](https://img.shields.io/badge/dataset-Zenodo%2013329934-blue.svg)](https://zenodo.org/records/13329934)

Analysis pipeline for the IEEE Access manuscript **"Side-Channel Attacks Survive Noise Cancellation in 3D Printers"** (Access-2026-31368) — a duration-controlled evaluation of acoustic and vibration side-channel leakage on FDM 3D printers equipped with Active Motor Noise Cancellation (AMNC).

AMNC is treated throughout as a **noise-reduction feature**. We located no patent, firmware documentation, or engineering statement connecting it to emanation suppression, and make no claim about vendor design intent.

---

## Findings

**Suppression is measurably operating**, yet leakage survives it. Spectral analysis shows the 120–360 Hz motor band sits only **+4.92 dB** above its background-relative baseline during printing — classifier-independent evidence, and the closest available substitute for the on/off ablation the hardware does not permit.

**Acoustic leakage is observation-time dependent, not eliminated.** At chance for 30-second windows (11.11%, permutation *p* = 0.188); **27.08%** at a duration-clean 60-second window; 40.28% under distributed sampling. Against an 8.33% baseline.

**Print duration is the dominant discriminator** — **63.89%** from recording length alone, validated against sliced G-code print time at *r* = 0.907. No acoustic countermeasure addresses it, and an adversary obtains it from start and stop times alone.

**Vibration carries geometry information independent of duration** (45.83% under an equalised observation window, *p* = 0.023) but adds nothing measurable beyond duration in paired comparison.

**A consumer handset performs as well as a mounted accelerometer** (25.00% vs 26.39%), and **leakage is architecture-specific** — present on the core-XY P1P, not distinguishable from chance on the bed-slinger A1 Mini.

---

## Results

Chance baseline 8.33% (1 of 12 classes), n = 144.

### Acoustic channel by observation window

| Audio observed | Accuracy | Recordings truncated |
|---|---|---|
| 10 s | 6.94% | 0 |
| 30 s | 11.11% | 0 |
| **60 s** | **27.08%** | **0** — duration-clean |
| 120 s | 30.56% | 20 |
| 300 s | 31.25% | 36 |
| 8 × 30 s distributed | 40.28% | — |

Simulated notch bank stays near baseline across quality factors (13.89 / 13.19 / 13.19% for Q = 15, 30, 60).

### Duration and vibration

| Condition | Accuracy | 95% CI |
|---|---|---|
| **Print duration alone** | **63.89%** | [55.78, 71.28] |
| Vibration, full recording | 29.17% | [22.36, 37.05] |
| Vibration, window equalised | 22.22% | [16.20, 29.68] |
| — amplitude only | 26.39% | [19.87, 34.13] |
| — frequency (Hz) only | 9.03% | [5.35, 14.83] |

G-code validation: Pearson *r* = 0.9073, Spearman ρ = 0.8933, n = 144.

### Duration controls (chance 25% within strata)

| Control | Condition | Accuracy |
|---|---|---|
| T1 | Duration only | 95.83% |
| T1 | Duration + vibration | 79.17% |
| T1 | Duration + acoustic | 56.25% |
| T1 | Duration + both | 50.00% |
| — | Fixed 60–120 s offset acoustic window | 31.25%, CI [19.95, 45.33] — includes chance |
| **T2** | **Vibration, window equalised to 543 s** | **45.83%**, *p* = 0.023 |
| T3 | Duration-overlap subset — **control failed** | 60.34% duration-only |

T1 shows sensor features never correct a duration-only error: the McNemar bottom-left cell is **zero** for every combination. T2 is the duration-equalised evidence of geometry-dependent vibration leakage. T3 was inconclusive and is not relied upon.

### Per capture configuration and per device

| Condition | Vibration accuracy | 95% CI |
|---|---|---|
| iPhone application (199.6 Hz) | 25.00% | [16.44, 36.09] |
| Teensy 4.0 accelerometer (500.0 Hz) | 26.39% | [17.59, 37.58] |
| Bambu P1P (core-XY) | 36.11% | [25.98, 47.65] |
| Bambu A1 Mini (bed-slinger) | 13.89% | [7.72, 23.71] — includes chance |
| Cross-printer transfer | 8.33% / 13.89% | at chance both directions |

Temporal model over equal-duration segments: 25.42% ± 2.31 ordered vs 15.28% ± 4.33 shuffled (five seeds).

---

## Dataset

> Madamopoulos, C. & Tsoutsos, N.G. (2024). *3D Printer Audio and Vibration Side Channel Dataset*. Zenodo. https://doi.org/10.5281/zenodo.13329934
>
> Described in: Madamopoulos & Tsoutsos, *Data in Brief* **57**, 111002 (2024).

- **Printers:** Bambu Lab P1P (core-XY) and A1 Mini (bed-slinger), both AMNC-equipped
- **Objects:** 12 classes · **Recordings:** 144 synchronised audio–vibration pairs (72 per printer)
- **Capture rigs:** iPhone application (72 recordings) and Teensy 4.0 with mounted accelerometer (72), balanced 6/6 per class
- **Ground truth:** sliced G-code included, used to validate print duration
- **Durations:** audio 88.3–11,273.3 s, vibration 116.9–11,265.5 s

The notebook downloads the dataset from Zenodo automatically.

---

## Notebook structure

`3d_printer_sidechannel_poc.ipynb` — each stage is **standalone**: it rebuilds its own catalog, extracts its own features, and writes structured JSON. Stages run independently, in any order. Stored outputs are preserved, so the printed values match the manuscript without re-running.

| Stage | Produces | Manuscript |
|---|---|---|
| 1 · Acoustic | window sweep, truncation audit, notch sweep, permutation tests | §VI-A, Table 2, Fig. 1 |
| 2 · Main analysis | duration, vibration, per-configuration, per-printer, transfer, temporal | §VI-B/C/E/F/G |
| 3 · G-code ground truth | recording duration vs sliced print time | §VI-B, Fig. 2 |
| 4 · Spectral verification | motor-band suppression vs background (+ exploratory) | §VI-H, Fig. 5 |
| 5 · Duration controls | T1, fixed-offset window, T2, T3 | §VI-D, Table 3 |
| 6 · Figures | the five manuscript figures | Figs. 1–5 |

Stage 4 also contains exploratory analyses (duration-stratified classification, acoustic window position, per-configuration acoustic split). These motivated Stage 5 and are kept for transparency; **only the spectral result is reported in the manuscript.**

```
├── 3d_printer_sidechannel_poc.ipynb
└── figures/
    ├── fig1_acoustic_window.png       # accuracy vs observation window
    ├── fig2_gcode_validation.png      # duration vs G-code print time
    ├── fig3_amplitude_vs_shape.png    # feature ablation (hertz)
    ├── fig4_per_device.png            # per-printer accuracy + transfer
    └── fig5_spectrum.png              # motor band vs background PSD
```

---

## Methods

### Pairing and cross-validation

Each audio file is paired with the accelerometer capture in its own session folder, giving 144 pairs; each recording is one labeled sample. The 144 recordings occupy 144 distinct sessions — one sample per session — so grouped and stratified cross-validation are equivalent here. The protocol is **stratified five-fold cross-validation**. Catalog composition (144 / 12 / 2) is verified programmatically and the analysis aborts on mismatch.

### Features

| Channel | Features | Dim. |
|---|---|---|
| Acoustic | 13 MFCCs + spectral centroid/bandwidth/rolloff/ZCR (mean + std) | 32 |
| Vibration (summary) | mean, std, RMS, peak-to-peak, dominant frequency **in Hz** × 3 axes | 15 |
| Vibration (temporal) | per-segment std/RMS/p2p/dominant-freq × 3 axes, 120 equal-**duration** segments | 120 × 12 |

Observation window is a controlled variable, not a fixed parameter. Vibration is truncated to a common window **in seconds**, not samples, because the two capture rigs run at different rates. Dominant frequency uses each recording's measured sampling rate — normalised-frequency features are not comparable across rigs.

### Statistics

- Wilson confidence intervals for all proportions
- Label-permutation null: **B = 1000**, full CV pipeline re-executed inside each permutation, estimator (r+1)/(B+1) bounded below by 0.001, one-sided
- Exact McNemar tests on paired predictions where two feature sets are compared on identical folds
- Order-shuffle control for the temporal model
- Deterministic data ordering, fixed random seeds, fixed cuDNN algorithm selection

### Known bounds

**Observable bandwidth.** Measured rates of 500.0 Hz and 199.6 Hz give Nyquist limits of 250 Hz and 99.5 Hz. Structural resonances in FDM printers occur above these. The frequency-at-chance result therefore bounds what was *observable*, not what exists.

**Duration matching is by observation window.** T2 equalises the window in seconds; it does not verify that no residual duration information survives in the truncated features. That error would reduce, not inflate, vibration's apparent independent contribution.

**G-code ground truth.** The sliced files shipped with the dataset are from the P1P; those print times were mapped onto A1 Mini recordings too, which likely attenuates the observed correlation.

---

## Reproduction

### Google Colab (recommended)

1. Open the notebook in Colab (badge above)
2. Mount Google Drive and run the setup cells — the dataset downloads from Zenodo to `MyDrive/3dprinter_sidechannel/`
3. Run any stage; each is standalone

A GPU runtime is needed only for the temporal model in Stage 2.

### Local

```bash
git clone https://github.com/ericyoc/3d_printer_sidechannel_poc
cd 3d_printer_sidechannel_poc
pip install librosa scikit-learn pandas numpy matplotlib scipy statsmodels torch
# Download the dataset from https://zenodo.org/records/13329934
# Set DRIVE_BASE in each stage's CONFIG block to your dataset path
jupyter notebook 3d_printer_sidechannel_poc.ipynb
```

---

## Citation

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
