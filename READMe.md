# 🩺 Blood Pressure Prediction from PPG Signals with Corrupted Segment Reconstruction

## 📌 Overview

This project presents a novel data processing and machine learning pipeline designed to predict **Mean Blood Pressure (mBP)** from **Photoplethysmography (PPG)** signals in anesthetized patients undergoing spine surgery. The project focuses on detecting and reconstructing corrupted segments in PPG data to improve prediction accuracy in a non-invasive, continuous monitoring setting.

Developed as part of a final B.Sc. project in Digital Medical Technologies, in collaboration with **Beilinson Hospital** and the **Nervio** digital health startup.

---

## 🎯 Project Objectives

- Detect and classify corrupted segments in PPG signals.
- Restore damaged data using **B-Spline** and **Cubic Spline** interpolation techniques.
- Extract meaningful time-series features from reconstructed signals.
- Train ML/DL models to predict mBP within ±5 mmHg accuracy (AAMI standard).
- Evaluate impact of signal reconstruction on prediction performance.

---

## 🛠 Tools & Technologies

- **Python**, NumPy, Pandas
- **SciPy** – signal processing & spline interpolation
- **Matplotlib / Seaborn** – visualization
- **Scikit-learn** – model development
- **Jupyter Notebooks / Google Colab**
- **PPG Dataset**: 67 patients, 2021–2022, from Beilinson Hospital

---

## 🧠 Methodology

### 1. Data Preprocessing
- Imported raw `.pkl` PPG signals & blood pressure data.
- Identified corrupted segments:
  - Saturation values (e.g. 1, 3276, 62258, 65535)
  - Repeated values
  - Missing timestamps
  - Low perfusion index (PI < 1)
- Treated corrupted PLETH values as `NaN` and removed short/noisy segments.

### 2. Synthetic Damage Generation
- Introduced artificial "holes" into clean PPG segments:
  - Two approaches: **Uniform** & **Realistic** distributions
  - Tested interpolation techniques on over 270,000 synthetic holes

### 3. Interpolation & Evaluation
- Applied **B-Spline** and **Cubic Spline** interpolation
- Assessed via **RMSE** and custom **Imputation Score**
- B-Spline proved more accurate on short gaps; Cubic Spline was better for longer gaps

### 4. Feature Extraction
- Extracted physiologically relevant features:
  - Wave duration
  - Time from start to systolic peak
  - Peak amplitudes, etc.
- Performed normalization and dimensionality reduction (PCA)

### 5. Model Training
- Trained several ML/DL models:
  - Neural Networks (preferred, if data allows)
  - Decision Trees (fallback)
- Evaluated via cross-validation and accuracy thresholds

---

## 📈 Results

- Created over **13,000 valid signal segments** for model training
- Achieved **~80% improvement in signal reconstruction** using B-Spline
- Built a complete framework for BP prediction using only a **single oximeter-based PPG** device
- Findings show that **reconstructing** rather than **discarding** corrupted segments preserves valuable information and enhances model performance

---

## 📚 Research Context

This project is grounded in recent medical literature on cuffless BP prediction and signal processing. Key references include:

- Slapnicar & Lustrek (2017), *Continuous BP Estimation from PPG*
- Chu et al. (2023), *Deep Learning Framework for Non-Invasive BP and SpO2 Estimation*
- Multiple studies on interpolation & PPG signal cleaning (e.g., cubic & B-spline methods)

---

## 🔬 Limitations & Future Work

- Dataset size (67 patients) may limit deep learning generalization
- Some interpolated gaps negatively impacted prediction accuracy
- Future directions:
  - Integrate additional physiological signals (ECG)
  - Expand dataset via public sources (e.g. MIMIC-IV)
  - Fine-tune DL models on per-patient basis

---

## 👥 Team

- **Idan Lichter** – Data analysis, signal processing, model training  
- **Adi Orpaz Idan** – Data engineering, methodology development, evaluation  
- Academic Supervisor: Dr. Dimitry Goldstein  
- Professional Supervisor: Mr. Yakir Menachem



