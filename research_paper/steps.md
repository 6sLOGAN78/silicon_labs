# Reproducing VitalML on VitalDB

## Overview

This project aims to reproduce the methodology of **VitalML** from the paper:

> Predicting Patient Decompensation from Continuous Physiologic Monitoring in the Emergency Department

using the **VitalDB** dataset.

Although the original study was conducted on Emergency Department (ED) patients at Stanford, VitalDB contains many of the same physiologic signals:

* ECG waveform
* PPG waveform
* Heart Rate (HR)
* Oxygen Saturation (SpO₂)
* Blood Pressure (MAP)

This makes VitalDB an excellent dataset for reproducing and extending the methodology.

---

# Important Difference Between Stanford ED and VitalDB

The original paper uses:

* Emergency Department patients
* Triage information
* Chief complaint
* Emergency Severity Index (ESI)

VitalDB primarily contains:

* Operating Room patients
* Continuous physiologic monitoring
* Waveform data
* Clinical metadata

Therefore:

✅ Waveform-based features can be reproduced.

❌ Chief complaint and ESI cannot be reproduced.

---

# Original VitalML Pipeline

## Inputs

### Static Features

* Age
* Gender
* Initial vital signs
* Chief complaint
* ESI score

### Continuous Features

Computed from the first 15 minutes of monitoring:

* Heart rate trend
* Respiratory rate trend
* SpO₂ trend
* MAP trend

### Engineered Waveform Features

#### ECG Features

* SDRR
* RMSSD
* pRR50
* TINN
* LF Power
* HF Power
* LF/HF Ratio

#### ECG + PPG Features

* Pulse Arrival Time (PAT)

#### PPG Features

* Perfusion Index (PI)

### Deep Features

Transformer embeddings from:

* 60-second ECG segment
* 60-second PPG segment

---

# Outputs

The paper trains separate models to predict:

## Tachycardia

```text
HR > 110 bpm
```

## Hypotension

```text
MAP < 65 mmHg
```

## Hypoxia

```text
SpO₂ < 90%
```

within the prediction window.

---

# Reproduction Strategy on VitalDB

---

# Step 1 — Select Valid Cases

Required signals:

```text
SNUADC/ECG_II
SNUADC/PLETH
Solar8000/HR
Solar8000/PLETH_SPO2
Solar8000/ART_MBP
```

Find all cases containing these tracks.

Example:

```python
TRACKS = [
    "SNUADC/ECG_II",
    "SNUADC/PLETH",
    "Solar8000/HR",
    "Solar8000/PLETH_SPO2",
    "Solar8000/ART_MBP"
]

caseids = vitaldb.find_cases(TRACKS)
```

Current VitalDB count:

```text
3506 cases
```

---

# Step 2 — Define Assessment and Prediction Windows

## Assessment Window

First 15 minutes:

```text
0 – 15 min
```

Used for feature extraction.

## Prediction Window

Next 90 minutes:

```text
15 – 105 min
```

Used for label generation.

Visual representation:

```text
|------ Assessment ------|--------- Prediction ----------|

0 min                 15 min                          105 min
```

---

# Step 3 — Create Labels

## Tachycardia

Positive if:

```python
max(HR_future) > 110
```

---

## Hypotension

Positive if:

```python
min(MAP_future) < 65
```

---

## Hypoxia

Positive if:

```python
min(SpO2_future) < 90
```

---

# Step 4 — Initial Stability Filtering

Only include patients who are initially stable.

Criteria:

```python
stable = (
    max(hr_first15) <= 110
    and
    min(spo2_first15) >= 90
    and
    min(map_first15) >= 65
)
```

Discard all cases that violate these conditions.

---

# Step 5 — Vital Sign Trend Features

Compute from the assessment window.

## Heart Rate

```python
mean_hr
std_hr
slope_hr
```

## Oxygen Saturation

```python
mean_spo2
std_spo2
slope_spo2
```

## Mean Arterial Pressure

```python
mean_map
std_map
slope_map
```

Linear trend slopes are calculated using least-squares regression.

---

# Step 6 — HRV Features

Extract from ECG.

Recommended library:

```text
NeuroKit2
```

Features:

```text
SDRR
RMSSD
pRR50
TINN
LF Power
HF Power
LF/HF Ratio
```

These reproduce the paper's ECG-derived HRV features.

---

# Step 7 — Pulse Arrival Time (PAT)

PAT is computed from ECG and PPG.

Definition:

```text
PAT =
PPG Peak Time
-
ECG R-Peak Time
```

Implementation:

```python
PAT = mean(
    PPG_peak_time -
    ECG_r_peak_time
)
```

Feature examples:

```python
pat_mean
pat_std
pat_min
pat_max
```

---

# Step 8 — Perfusion Index (PI)

Derived from PPG.

Formula:

```text
PI = AC / DC
```

where:

```text
AC = Pulsatile component
DC = Baseline component
```

Feature examples:

```python
pi_mean
pi_std
pi_min
pi_max
```

The original paper identified PI as one of the most important predictors for hypoxia.

---

# Step 9 — Baseline Model

Do NOT start with transformers.

Build a baseline first.

Features:

```text
Vital Sign Trends
+
HRV
+
PAT
+
Perfusion Index
```

Model:

```python
lightgbm.LGBMClassifier
```

Separate models for:

```text
Tachycardia
Hypotension
Hypoxia
```

Evaluate:

```text
AUROC
AUPRC
Sensitivity
Specificity
```

---

# Step 10 — Deep Learning Extension

After baseline reproduction succeeds:

## ECG Encoder

Input:

```text
60-second ECG waveform
```

Output:

```text
Embedding vector
```

---

## PPG Encoder

Input:

```text
60-second PPG waveform
```

Output:

```text
Embedding vector
```

---

## Final Model

```text
Vital Trends
+
HRV
+
PAT
+
Perfusion Index
+
ECG Embedding
+
PPG Embedding
```

↓

```text
LightGBM
```

↓

```text
Tachycardia
Hypotension
Hypoxia
```

---

# Recommended Project Structure

```text
vitalml_vitaldb/

├── data/
│   ├── raw/
│   └── processed/
│
├── src/
│   ├── dataset/
│   │   ├── load_vitaldb.py
│   │   ├── create_windows.py
│   │   └── labels.py
│   │
│   ├── features/
│   │   ├── trends.py
│   │   ├── hrv.py
│   │   ├── pat.py
│   │   └── perfusion.py
│   │
│   ├── models/
│   │   ├── lightgbm_train.py
│   │   └── evaluate.py
│   │
│   └── transformers/
│       ├── ecg_encoder.py
│       └── ppg_encoder.py
│
└── notebooks/
    └── exploration.ipynb
```

---

# Development Order

1. Case Selection
2. Assessment Window Generation
3. Label Creation
4. Trend Features
5. HRV Features
6. PAT Features
7. Perfusion Index
8. Baseline LightGBM
9. Model Evaluation
10. Transformer Embeddings
11. Full VitalML Reproduction
12. Research Extensions

```
```
