# PPG Blood Pressure Prediction

A machine learning pipeline that predicts blood pressure from photoplethysmography (PPG) signals — without a pressure cuff.

Developed as a B.Sc. final project at HIT (Holon Institute of Technology) in collaboration with Beilinson Hospital and the Nervio digital health startup.

---

## What This Project Does

A standard pulse oximeter clips to your finger and produces a PPG signal — the same waveform used to measure heart rate. This project investigates whether that signal alone contains enough information to also predict **blood pressure** (MAP, SBP, and DBP), using only signal processing and classical ML.

Key contributions:
- A robust preprocessing pipeline that **reconstructs corrupted PPG segments** (saturation artefacts, flat-lines) instead of discarding them — preserving significantly more usable data
- Seven clinically motivated **feature groups** extracted from each cardiac cycle
- A systematic comparison of **4 ML models × 4 hyperparameter grids × 6 BP targets**

---

## Dataset

- **67 patients** undergoing spine surgery under general anesthesia at Beilinson Hospital (2021–2022)
- PPG recorded at **100 Hz** via pulse oximeter
- Simultaneous invasive BP from an arterial line (ground truth)
- Data is **not included** in this repository (patient privacy). Place your `.pkl` files in `data/raw/train/` and `data/raw/test/` following the naming convention `BP_Table_<id>.pkl` / `PPG_Table_<id>.pkl`

---

## Pipeline Overview

```
data/raw/train/
    │
    ▼
01_data_import.ipynb          Load raw data, remove saturation artefacts
    │
    ▼
02_signal_processing.ipynb    Bandpass filter, SDET peak detection, quality windows
    │
    ▼
03_interpolation.ipynb        Synthetic gap creation + Cubic/B-Spline reconstruction
    │
    ▼
04_data_loading.ipynb         Organise train/test splits for ML
    │
    ▼
05_feature_extraction.ipynb   Extract 356 physiological features per 1-minute window
    │
    ▼
06_model_training.ipynb       GridSearchCV across 4 models x 6 BP targets → outputs/
```

All intermediate data is saved in `data/processed/` so each notebook can be run independently after its prerequisites.

---

## Notebooks

| Notebook | Purpose |
|----------|---------|
| [01_data_import](notebooks/improved/01_data_import.ipynb) | Load `.pkl` files, handle timestamp resets, remove saturation values |
| [02_signal_processing](notebooks/improved/02_signal_processing.ipynb) | Butterworth filter, SDET peak detection, quality window selection |
| [03_interpolation](notebooks/improved/03_interpolation.ipynb) | Synthetic gap generation, Cubic Spline vs B-Spline evaluation |
| [04_data_loading](notebooks/improved/04_data_loading.ipynb) | Align BP and PPG data for ML, train/test split |
| [05_feature_extraction](notebooks/improved/05_feature_extraction.ipynb) | Extract 7 feature groups from cardiac cycles |
| [06_model_training](notebooks/improved/06_model_training.ipynb) | SVR, Decision Tree, Random Forest with GridSearchCV |

The original notebooks are preserved in `notebooks/` for reference.

---

## Features Extracted (Notebook 05)

Each 1-minute PPG window yields **356 features** (mean, std, min, max per cycle-level measurement):

| Group | Feature | Clinical meaning |
|-------|---------|-----------------|
| Area | AUC per cycle | Blood vessel compliance, stroke volume |
| Width-Amplitude | Width at 25/50/75% of peak | Vascular resistance |
| ROR / ROF | Rate of Rise / Fall | Systolic upstroke, diastolic run-off |
| Cycle Time | Min-to-min interval | Heart rate |
| Y Height | Systolic peak amplitude | Pulse pressure proxy |
| STT | Slope / amplitude | Timing independent of amplitude |
| Total Variation | Sum of abs(delta signal) | Signal roughness / noise level |

---

## Models & Results

Four hyperparameter grids were evaluated for each of the 6 BP targets:

| Grid | Model | Key hyperparameters |
|------|-------|-------------------|
| Grid 1 | SVR — RBF | C in {0.1, 1, 10, 100}, gamma in {scale, auto} |
| Grid 2 | SVR — Polynomial | C, degree in {2,3,4}, gamma |
| Grid 3 | Decision Tree | max_depth in {5,10,15,20}, criterion |
| Grid 4 | Random Forest | n_estimators in {50,100,200}, max_depth |

All models share the same preprocessing pipeline: **impute → standardise → PCA (95% variance)**.

Cross-validation uses `GroupShuffleSplit` to ensure no patient leaks between train and validation folds.

Clinical target: **RMSE <= 5 mmHg** (AAMI standard for non-invasive BP monitors).

Trained models are saved to `outputs/models/`, result CSVs and plots to `outputs/results/`.

---

## Repository Structure

```
ppg-blood-pressure-prediction/
├── notebooks/
│   ├── improved/               <- Clean, documented notebooks (use these)
│   │   ├── 01_data_import.ipynb
│   │   ├── 02_signal_processing.ipynb
│   │   ├── 03_interpolation.ipynb
│   │   ├── 04_data_loading.ipynb
│   │   ├── 05_feature_extraction.ipynb
│   │   └── 06_model_training.ipynb
│   └── Final_MOP_*.ipynb       <- Original notebooks (preserved for reference)
├── data/
│   ├── raw/
│   │   ├── train/              <- BP_Table_*.pkl and PPG_Table_*.pkl (not in git)
│   │   └── test/
│   └── processed/              <- Auto-generated intermediate outputs (not in git)
├── outputs/
│   ├── models/                 <- Trained model .sav files
│   └── results/                <- CSV results and prediction plots
├── requirements.txt
└── README.md
```

---

## Getting Started

```bash
# 1. Clone the repository
git clone https://github.com/IdanLichter/ppg-blood-pressure-prediction.git
cd ppg-blood-pressure-prediction

# 2. Create a virtual environment and install dependencies
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 3. Add your data
# Copy patient .pkl files into data/raw/train/ and data/raw/test/
# Files must follow the naming: BP_Table_<patient_id>.pkl  /  PPG_Table_<patient_id>.pkl

# 4. Run notebooks in order
jupyter notebook notebooks/improved/
```

Run notebooks 01 to 06 in sequence. Each notebook saves its outputs to `data/processed/`, which the next notebook reads.

---

## Research Context

This project is grounded in recent medical literature on cuffless BP prediction and signal processing:

- Slapnicar & Lustrek (2017), *Continuous BP Estimation from PPG*
- Chu et al. (2023), *Deep Learning Framework for Non-Invasive BP and SpO2 Estimation*

---

## Limitations & Future Work

- Dataset size (67 patients) may limit deep learning generalisation
- Some interpolated gaps negatively impacted prediction accuracy in edge cases
- Future directions:
  - Integrate additional physiological signals (ECG)
  - Expand dataset via public sources (MIMIC-IV)
  - Fine-tune models on a per-patient basis

---

## Team

| Role | Name |
|------|------|
| Data analysis, signal processing, model training | Idan Lichter |
| Data engineering, methodology, evaluation | Adi Orpaz Idan |
| Academic supervisor | Dr. Dimitry Goldstein |
| Professional supervisor | Mr. Yakir Menachem |

**Institution:** Holon Institute of Technology (HIT) — B.Sc. Digital Medical Technologies, 2022–2023
