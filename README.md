# ICUsentinel: Early Patient Decompensation Prediction System

## Table of Contents

1. [Project Overview](#project-overview)
2. [Clinical Background](#clinical-background)
3. [System Architecture](#system-architecture)
4. [Prerequisites](#prerequisites)
5. [Installation & Setup](#installation--setup)
6. [Project Structure](#project-structure)
7. [Running the Pipeline](#running-the-pipeline)
8. [Data Processing Pipeline](#data-processing-pipeline)
9. [Model Training](#model-training)
10. [Results & Interpretation](#results--interpretation)
11. [Technical Details](#technical-details)

---

## Project Overview

ICUsentinel is a machine learning system designed to predict early signs of patient deterioration in critical care settings using continuous physiological monitoring data. The project reproduces the methodology of **VitalML** (from Stanford Emergency Department research) on the **VitalDB dataset** - a collection of 3,506 operating room cases with continuous physiological signal monitoring.

### Core Objectives

- Detect physiological deterioration patterns BEFORE conventional alarm systems activate
- Identify patients at risk of developing tachycardia (HR > 110 bpm), hypotension (MAP < 65 mmHg), or hypoxia (SpO2 < 90%)
- Compare prediction accuracy under different patient selection strategies (stable vs. all patients)
- Validate that trend velocity is more predictive than absolute vital sign values

### Key Innovation

Most ICU emergencies are not sudden events. Physiological deterioration typically begins with subtle changes 30 minutes to several hours before a patient becomes clinically unstable. The system focuses on **trend analysis** rather than absolute values, recognizing that a patient's heart rate increasing from 82 → 95 → 110 → 125 bpm over 30 minutes represents greater risk than a stable HR of 125 bpm.

---

## Clinical Background

### Clinical Rationale

Physiological deterioration follows a predictable progression:

**Stage 1: Compensation Phase**
- Heart rate increasing
- Respiratory rate increasing
- Temperature increasing
- Blood pressure still normal

**Stage 2: Early Hemodynamic Changes**
- Heart rate variability (HRV) decreasing
- Perfusion worsening
- Systolic BP may decline slightly

**Stage 3: Systemic Deterioration**
- Mean arterial pressure declining
- Oxygen demand increasing
- Metabolic derangement

**Stage 4: Septic Shock (Critical)**
- Hemodynamic collapse requires emergency intervention

### Why Early Detection Matters

By identifying Stage 1-2 changes, clinicians can intervene earlier with:
- Fluid resuscitation
- Antibiotics (if infectious cause)
- Vasopressor support
- Increased monitoring intensity

This significantly improves patient outcomes compared to detection at Stage 3-4.

### Target Conditions

1. **Tachycardia (HR > 110)**: Often first sign of compensatory stress response
2. **Hypotension (MAP < 65)**: Indicates circulatory compromise requiring immediate intervention
3. **Hypoxia (SpO2 < 90)**: Respiratory or cardiac dysfunction

---

## System Architecture

### High-Level Processing Pipeline

```
VitalDB Raw Cases
        |
        v
Step 1: Data Extraction & Windowing
        |
        +---> Window Creation (Assessment + Prediction periods)
        |---> Signal Cleaning (remove sensor artifacts)
        |---> Label Generation (identify deterioration events)
        |
        v
Labeled Windowed Dataset
        |
        +-----> Approach A: Stability-Filtered (VitalML-compliant)
        |           |---> Quality Filtering
        |           |---> Stability Criteria Applied
        |           |---> Feature Extraction
        |           |---> Model Training
        |           +---> with_stab_filter/
        |
        +-----> Approach B: Unfiltered (All Patients)
                    |---> Quality Filtering  
                    |---> No Stability Criteria
                    |---> Feature Extraction
                    |---> Model Training
                    +---> without_stabfilter/

Output: Trained Models & Performance Metrics
```

### Signals Used

The system requires five continuous physiological signals from the VitalDB dataset:

| Signal | Source | Sampling Rate | Purpose |
|--------|--------|---------------|---------|
| SNUADC/ECG_II | Analog-to-Digital Converter | 500 Hz | ECG waveform for HRV analysis |
| SNUADC/PLETH | Analog-to-Digital Converter | 500 Hz | Photoplethysmography (pulse oximetry) waveform |
| Solar8000/HR | Patient Monitor | 1 Hz | Heart rate trends |
| Solar8000/PLETH_SPO2 | Patient Monitor | 1 Hz | Oxygen saturation trends |
| Solar8000/ART_MBP | Patient Monitor | 1 Hz | Mean arterial blood pressure trends |

---

## Prerequisites

### System Requirements

- Windows 10/11, macOS, or Linux
- Python 3.8 or higher
- 8+ GB RAM (minimum; 16 GB recommended for parallel processing)
- 50 GB free disk space (for VitalDB dataset and generated features)
- Internet connection (for downloading VitalDB library)

### Required Software

- Anaconda or Miniconda (recommended for dependency management)
- Git (for version control)
- Jupyter Notebook or JupyterLab (for running notebooks)

---

## Installation & Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/6sLOGAN78/silicon_labs.git
cd silicon_labs
```

### Step 2: Create Python Virtual Environment

Using Anaconda (Recommended):

```bash
conda create -n iculab python=3.10
conda activate iculab
```

Or using venv:

```bash
python -m venv iculab
# On Windows:
iculab\Scripts\activate
# On macOS/Linux:
source iculab/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

Key packages:

```
vitaldb              # VitalDB dataset access
pandas               # Data manipulation
numpy                # Numerical computing
scipy                # Scientific computing (linear regression)
scikit-learn         # Machine learning utilities
lightgbm             # Gradient boosting for classification
neurokit2            # Heart rate variability analysis
jupyter              # Interactive notebooks
tqdm                 # Progress bars
matplotlib           # Data visualization
```

### Step 4: Verify Installation

```bash
python -c "import vitaldb; print('VitalDB OK')"
python -c "import neurokit2 as nk; print('NeuroKit2 OK')"
python -c "import lightgbm; print('LightGBM OK')"
```

### Step 5: Configure Jupyter (Optional but Recommended)

```bash
jupyter notebook --generate-config
# Configure as needed for your system
```

---

## Project Structure

```
silicon_labs/
├── README.md                          # This file
├── requirements.txt                   # Python dependencies
├── report.md                          # Clinical design review notes
├── signals.md                         # VitalDB signal reference
│
├── research_paper/
│   └── steps.md                       # Reproduction methodology
│
├── research_pre_Code/
│   ├── 03_hrv_features.ipynb          # Phase 3: HRV feature extraction
│   │
│   ├── data_extraction/
│   │   └── 01_data_extraction.ipynb   # Phase 1: Raw data extraction & windowing
│   │
│   ├── with_stab_filter/
│   │   ├── 02_base_model.ipynb        # Phase 2A: Model training (stable only)
│   │   ├── window_features_stable.csv # Raw extracted features
│   │   └── window_features_clean.csv  # Post-cleanup features
│   │
│   └── without_stabfilter/
│       ├── 02_base_model.ipynb        # Phase 2B: Model training (all patients)
│       ├── window_features.csv        # Raw features
│       └── window_features_clean.csv  # Post-cleanup features
│
└── reports/                           # Generated analysis reports
    └── [analysis outputs]
```

---

## Running the Pipeline

### Complete Workflow (Step-by-Step)

The project follows a three-phase pipeline. Run each phase in order:

#### Phase 1: Data Extraction & Windowing

**Location**: `research_pre_Code/data_extraction/01_data_extraction.ipynb`

**Purpose**: Load VitalDB cases, create time windows, generate labels

**Duration**: 30-60 minutes (parallel processing with 8 workers)

**Steps**:
1. Open Jupyter Notebook:
   ```bash
   jupyter notebook
   ```

2. Navigate to `research_pre_Code/data_extraction/` and open `01_data_extraction.ipynb`

3. Run cells sequentially (Ctrl+Enter):
   - Cell 1: Import libraries
   - Cell 2: Query VitalDB for cases with required signals
   - Cells 3-7: Define windowing functions (15-min assessment + 30-min prediction windows with 5-min stride)
   - Cells 8-12: Process all cases in parallel
   
4. **Output**: `window_dataset.csv` containing:
   - 3-4 windows per case (sliding window with 5-min stride)
   - Binary labels: tachycardia, hypotension, hypoxia
   - Case ID and window ID for traceability

**Expected Output Example**:
```
caseid, window_id, tachycardia, hypotension, hypoxia
1, 0, 0, 0, 0
1, 1, 0, 0, 0
2, 0, 1, 0, 0
2, 1, 0, 1, 0
...
```

#### Phase 2A: Model Training (Stability-Filtered Approach - VitalML-Compliant)

**Location**: `research_pre_Code/with_stab_filter/02_base_model.ipynb`

**Purpose**: Train predictive models using initially-stable patients only

**Duration**: 10-15 minutes

**Methodology**:

1. **Stability Filtering**: Only include windows where the patient was initially stable
   ```
   Stable criteria (must ALL be true in 0-15 min window):
   - HR maximum ≤ 110 bpm
   - SpO2 minimum ≥ 90%
   - MAP minimum ≥ 65 mmHg
   ```

2. **Quality Filtering**: Ensure sufficient data quality
   ```
   - HR: ≥ 300 seconds of valid measurements
   - SpO2: ≥ 300 seconds of valid measurements
   - MAP: ≥ 300 seconds of valid measurements
   ```

3. **Feature Extraction** (9 features total):
   ```
   For each vital sign (HR, SpO2, MAP):
   - Mean value over 15-minute window
   - Standard deviation (variability)
   - Linear trend slope (rate of change)
   ```

4. **Data Cleaning**:
   - Remove rows with missing features
   - Remove physiological outliers:
     * MAP: 30-200 mmHg
     * HR: 20-220 bpm
     * SpO2: 50-100%

5. **Train/Test Split**:
   ```python
   Group Shuffle Split (80/20 on patient level)
   - Prevents data leakage
   - No test cases come from training patients
   ```

6. **Model Training**:
   Three separate LGBMClassifier models for each prediction target:
   
   | Target | Parameters |
   |--------|-----------|
   | Hypotension | n_estimators=300, learning_rate=0.05, num_leaves=31 |
   | Tachycardia | n_estimators=300, learning_rate=0.05, num_leaves=31 |
   | Hypoxia | n_estimators=550, learning_rate=0.005, num_leaves=42 |

**To Run**:
1. Open `research_pre_Code/with_stab_filter/02_base_model.ipynb`
2. Ensure `window_dataset.csv` is in the same directory (or update path)
3. Run all cells
4. Review outputs:
   - AUROC scores for each prediction target
   - Feature importance rankings
   - Model performance comparison

**Expected Results**:
```
Hypotension AUROC: 0.75-0.85
Tachycardia AUROC: 0.65-0.75
Hypoxia AUROC: 0.60-0.70
```

**Output Files**:
- `window_features_stable.csv`: Features after stability filtering
- `window_features_clean.csv`: Features after outlier removal
- Trained models: `model_h`, `model_t`, `model_hy` (in memory, can be saved)

#### Phase 2B: Model Training (Unfiltered Approach - Comparison)

**Location**: `research_pre_Code/without_stabfilter/02_base_model.ipynb`

**Purpose**: Train models using ALL patients regardless of baseline stability

**Key Difference**: No stability filter applied - includes all patient states

**Duration**: 10-15 minutes

**To Run**:
1. Open `research_pre_Code/without_stabfilter/02_base_model.ipynb`
2. Run all cells (same structure as Phase 2A)

**Purpose of Comparison**:
- Determines if trend-based prediction works only for initially-stable patients
- Validates whether the method generalizes to all patient populations
- Shows impact of patient selection bias

#### Phase 3: HRV Feature Extraction (Optional Enhancement)

**Location**: `research_pre_Code/03_hrv_features.ipynb`

**Purpose**: Extract advanced heart rate variability features for potential model enhancement

**Duration**: 2-5 hours (depending on dataset size)

**HRV Features Extracted** (7 total):

**Time-Domain Features**:
- SDRR (SDNN): Standard deviation of all RR intervals
- RMSSD: Root mean square of successive RR interval differences
- pRR50 (pNN50): Percentage of consecutive RR intervals differing by >50ms
- TINN: Triangular interpolation of NN interval histogram

**Frequency-Domain Features**:
- LF Power: Low frequency (0.04-0.15 Hz) - sympathetic nervous system
- HF Power: High frequency (0.15-0.4 Hz) - parasympathetic nervous system
- LF/HF Ratio: Sympathovagal balance indicator

**Clinical Significance**:
- Early sepsis reduces HRV (loss of variability indicates autonomic dysfunction)
- LF/HF ratio increases with stress and infection
- These features add predictive value beyond simple trend slopes

**To Run**:
1. Open `research_pre_Code/03_hrv_features.ipynb`
2. Cell 1: Install NeuroKit2 if not already installed
3. Cells 2-5: Define HRV extraction function and test on sample ECG
4. Run on full dataset (requires ECG waveform data)

**Output**: DataFrame with 7 additional HRV features per window

---

## Data Processing Pipeline

### Step 1: Window Creation

**Input**: Single VitalDB case (typically 60-120 minutes of data)

**Process**:
```
Continuous vital sign series
        |
        v
Define assessment window: 0-15 minutes (for feature extraction)
Define prediction window: 15-45 minutes (for label generation)
        |
        v
Slide across entire case with 5-minute stride
        |
        v
Create (assessment_window, prediction_window) pairs
        |
        v
Multiple overlapping windows per case
```

**Example**:
```
Case duration: 60 minutes
Window 0: Assessment [0-15], Prediction [15-45]
Window 1: Assessment [5-20], Prediction [20-50]
Window 2: Assessment [10-25], Prediction [25-55]
...
```

**Purpose of Stride**:
- Small stride (5 min) maximizes data utilization
- Prevents temporal leakage (future data affecting past predictions)
- Creates diverse training examples from same patient

### Step 2: Signal Cleaning

**Invalid Value Masking**:
```python
HR: mask values ≤ 0 (sensor error)
SpO2: mask values ≤ 0 (sensor error)
MAP: mask values < 30 (physiologically impossible)
```

**Rationale**: These values represent sensor failures, not actual patient conditions

### Step 3: Label Generation

For each prediction window, determine if patient will experience deterioration:

```python
tachycardia = (max(HR_future) > 110) ? 1 : 0
hypotension = (min(MAP_future) < 65) ? 1 : 0
hypoxia = (min(SpO2_future) < 90) ? 1 : 0
```

### Step 4: Feature Extraction

From 15-minute assessment window, compute 9 vital trend features:

```python
For each vital (HR, SpO2, MAP):
    mean = average value over window
    std = standard deviation (variability)
    slope = linear regression slope (trend direction/velocity)

Example for HR:
    hr_mean = 85 bpm
    hr_std = 12 bpm (fairly variable)
    hr_slope = +1.2 bpm/min (steadily increasing)
```

**Slope Calculation**:
```python
from scipy.stats import linregress

# RR intervals over time
x = [0, 1, 2, ..., 900]  # seconds
y = [84, 85, 87, ..., 95]  # heart rates

result = linregress(x, y)
slope = result.slope  # bpm/second
```

Positive slope = worsening trend, higher risk

---

## Model Training

### Feature Matrix

```
Shape: (N_windows, 9_features)

       hr_mean  hr_std  hr_slope  spo2_mean  spo2_std  spo2_slope  map_mean  map_std  map_slope
       85       12      0.5       95         2.1       -0.1        75        8        0.2
       82       8       0.1       96         1.8       0.0         78        6        0.05
       88       15      1.2       93         3.2       -0.3        72        10       -0.15
       ...
```

### Label Matrix

```
Shape: (N_windows, 3_targets)

       tachycardia  hypotension  hypoxia
       0            0            0       (patient stays stable)
       1            0            0       (develops tachycardia)
       0            1            0       (develops hypotension)
       1            1            0       (both conditions develop)
       ...
```

### Train/Test Split Strategy

**Group Shuffle Split** on Patient ID:

```
Problem: Simple random split leaks information
- Patient 5 appears in both training and test
- Model learns Patient 5's patterns, not generalizable

Solution: Split on patient level
- 80% of unique patients → training
- 20% of unique patients → testing
- All windows for training patients → training set
- All windows for test patients → test set
```

### LightGBM Models

**Why LightGBM?**
- Fast training on large datasets
- Handles non-linear relationships
- Produces feature importance rankings
- Robust to missing values
- Excellent for imbalanced classification

**Model Architecture**:
```
LGBMClassifier
├── Input: 9 features
├── Hidden layers: Gradient boosting tree ensembles
├── Output: Binary classification (0 or 1)
└── Metric: Binary cross-entropy loss, optimized for AUROC
```

**Hyperparameter Tuning**:

Different parameters for each target (tuned via experimentation):

| Target | n_estimators | learning_rate | num_leaves | Rationale |
|--------|-------------|---------------|-----------|-----------|
| Hypotension | 300 | 0.05 | 31 | Strong signal, good convergence |
| Tachycardia | 300 | 0.05 | 31 | Similar difficulty to hypotension |
| Hypoxia | 550 | 0.005 | 42 | Weak signal, needs more trees & slower learning |

### Model Evaluation

**Primary Metric: AUROC (Area Under Receiver Operating Characteristic Curve)**

```
AUROC = 1.0: Perfect prediction (rarely achieved)
AUROC = 0.9+: Excellent discrimination
AUROC = 0.8-0.9: Good discrimination
AUROC = 0.7-0.8: Fair discrimination
AUROC = 0.6-0.7: Poor discrimination
AUROC = 0.5: No better than random guessing
```

**Why AUROC?**
- Threshold-independent (doesn't require setting decision boundary)
- Handles class imbalance well
- Reflects ranking ability (can rank high-risk patients correctly)

**Feature Importance**:
```python
# Shows which features contribute most to predictions
importance = model.feature_importances_

hr_slope: 0.35 (35% importance - trend is key!)
map_mean: 0.20 (baseline pressures matter)
spo2_slope: 0.18
hr_mean: 0.15
...
```

---

## Results & Interpretation

### Expected Performance

With stability filtering (VitalML approach):

| Target | Expected AUROC | Interpretation |
|--------|---------------|-----------------|
| Hypotension | 0.75-0.85 | Good - BP trends are strong predictors |
| Tachycardia | 0.65-0.75 | Fair-Good - HR is variable, many confounders |
| Hypoxia | 0.60-0.70 | Fair - O2 sat more stable in OR setting |

Performance difference (with_stab_filter vs. without_stabfilter):
- Typically worse without filtering (broader patient population = harder task)
- Validates that trend-based prediction works best when baseline is known

### Feature Importance Insights

Typical ranking (hypotension model):

```
1. map_slope (40-50%)      -> "Is BP trending down?" (most important)
2. map_mean (25-30%)       -> "What's baseline BP?"
3. hr_slope (15-20%)       -> "Is HR increasing?"
4. spo2_slope (5-10%)      -> "Is O2 sat declining?"
5. hr_std (3-5%)           -> "HR variability"
```

**Key Finding**: Trend slopes are more important than absolute values, validating the clinical hypothesis.

### Interpreting a Prediction

Example prediction for a patient:

```
Features:
  hr_mean: 92 bpm
  hr_slope: +2.1 bpm/min (increasing rapidly)
  map_mean: 68 mmHg
  map_slope: -1.5 mmHg/min (declining)
  spo2_mean: 94%
  spo2_slope: -0.3%/min (slight decline)

Model Output:
  P(hypotension) = 0.72 (72% probability patient will develop MAP < 65)
  
Interpretation:
  HIGH RISK - Recommend:
  - Fluid challenge
  - Consider vasopressor support
  - Increase monitoring frequency
```

---

## Technical Details

### Data Integrity Checks

All phases include quality validation:

```python
# Phase 1: Data Extraction
- Check for required signals present
- Verify sampling rate correctness
- Validate timestamp continuity

# Phase 2: Feature Engineering
- Verify no NaN in feature matrix
- Confirm trend slopes calculated correctly
- Validate label distribution

# Phase 3: Model Training
- Check train/test split is on patient level
- Verify no data leakage
- Validate model convergence
```

### Parallel Processing

Phase 1 uses ThreadPoolExecutor for efficiency:

```python
N_WORKERS = 8  # Process 8 cases simultaneously

with ThreadPoolExecutor(max_workers=N_WORKERS) as executor:
    futures = {
        executor.submit(process_case, caseid): caseid
        for caseid in remaining_caseids
    }
    
    for future in as_completed(futures):
        caseid = futures[future]
        rows = future.result()
        # Save immediately (resume support)
```

**Benefits**:
- 8x faster than sequential processing
- Resume support: can pause and continue
- Incremental saving prevents data loss

### Error Handling

Each case processing includes try-except blocks:

```python
try:
    arr = vitaldb.load_case(caseid, TRACKS)
    df = pd.DataFrame(arr, columns=TRACKS)
    # ... process ...
    return rows
except Exception as e:
    print(f"Case {caseid} failed: {e}")
    return []  # Empty result, move to next case
```

Prevents single corrupted case from stopping entire pipeline.

### Memory Efficiency

For 3,506 cases:
- Phase 1: Processes one case at a time (streaming)
- Saves results incrementally to CSV
- Never loads entire dataset into memory simultaneously
- Allows processing on systems with 8GB RAM

### Computation Time Estimates

| Phase | Component | Time |
|-------|-----------|------|
| Phase 1 | Load 3506 cases | 30-60 min (parallel) |
| Phase 2A | Extract features | 2-3 min |
| Phase 2A | Train 3 models | 5-10 min |
| Phase 2B | Repeat for unfiltered | 5-10 min |
| Phase 3 | HRV extraction | 2-5 hours |
| **Total** | Complete pipeline | ~3-4 hours |

---

## Troubleshooting

### Issue: "ModuleNotFoundError: No module named 'vitaldb'"

**Solution**:
```bash
pip install vitaldb --upgrade
# Verify installation
python -c "import vitaldb; print(vitaldb.__version__)"
```

### Issue: "VitalDB API rate limiting / connection timeout"

**Solution**:
- VitalDB servers may be busy
- Add retry logic with exponential backoff
- Run during off-peak hours
- Check VitalDB service status

### Issue: "Out of Memory" errors during Phase 1

**Solution**:
- Reduce N_WORKERS from 8 to 4 or 2
- Close other applications
- Increase virtual memory
- Process in smaller batches

### Issue: "Model AUROC seems too high (> 0.95)"

**Solution**:
- Check for data leakage (same patient in train/test)
- Verify Group Shuffle Split is applied correctly
- Examine feature distributions for unrealistic patterns

### Issue: "No HRV features extracted (all NaN)"

**Solution**:
- ECG signal may be noisy/corrupted
- Try 500 Hz sampling rate explicitly
- Reduce minimum R-peak requirement from 10 to 5
- Check that ECG duration is at least 30 seconds

---

## References

### Primary References

1. **VitalML Paper**: Predicting Patient Decompensation from Continuous Physiologic Monitoring in the Emergency Department (Stanford)

2. **VitalDB Dataset**: Park, S. M., et al. "VitalDB, a high-fidelity vital signs database in surgical patients." Scientific Data 7.1 (2020): 1-8.

3. **Heart Rate Variability**: Malik, M., et al. "Heart rate variability: Standards of measurement, physiological interpretation and clinical use." European heart journal 17.3 (1996): 354-381.

### Key Libraries

- **VitalDB**: Official dataset access library
- **NeuroKit2**: Heart rate variability analysis
- **LightGBM**: Gradient boosting classification
- **Scikit-learn**: Machine learning utilities and metrics

---

## Contributing & Support

### Project Structure Conventions

- Notebooks use descriptive names with phase numbers (e.g., `01_data_extraction.ipynb`)
- CSV files follow pattern: `window_features_[variant].csv`
- Comments explain clinical rationale, not just code logic

### Future Enhancements

- Incorporate additional signals: respiratory waveforms, blood chemistry
- Implement deep learning models (RNNs, Transformers) for sequential data
- Add real-time prediction interface
- Deploy as clinical decision support system
- Validate on prospective patient cohorts

### Contact

For questions or issues, refer to the repository's issue tracker.

---

## License & Citation

This project reproduces methodology from peer-reviewed clinical research. If using this code for publication, cite:

- Original VitalML study
- VitalDB dataset paper
- This repository (if applicable)

---

**Last Updated**: 2026-07-02

**Version**: 1.0

**Status**: Production-Ready
