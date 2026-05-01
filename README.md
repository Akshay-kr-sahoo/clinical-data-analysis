# 🏥 Clinical Data Cleaning & Analysis using Python

> A Python-based data pipeline that cleans, validates, and transforms a mock clinical trial dataset (SDTM-like structure) with 5,000+ records — built as an academic/personal project to demonstrate clinical data management and CDISC SDTM concepts.

---

## 📌 Project Overview

This project simulates the real-world workflow of a **Clinical Data Manager or Clinical Programmer** — cleaning raw clinical trial data, detecting anomalies, mapping data into CDISC SDTM domains, and generating visual summary reports. It is built entirely in Python using Google Colab / Jupyter Notebook.

| Attribute        | Detail                                      |
|------------------|---------------------------------------------|
| **Domain**       | Clinical Data Management / Biostatistics    |
| **Dataset Size** | 5,000+ records                              |
| **Dataset Type** | Mock Clinical Trial (SDTM-like structure)   |
| **Environment**  | Google Colab / Jupyter Notebook             |
| **Tools Used**   | Python, Pandas, NumPy, Matplotlib, MS Excel |

---

## 🗂️ Dataset Description

The dataset (`clean_clinical_data.csv`) is structured similarly to CDISC SDTM and contains the following columns:

| Column     | Description                                      | Example Values              |
|------------|--------------------------------------------------|-----------------------------|
| `SUBJID`   | Unique Subject Identifier                        | 8, 10, 11, 12 ...           |
| `AGE`      | Subject Age (years)                              | 28, 40, 56 ...              |
| `SEX`      | Subject Sex                                      | M, F                        |
| `AE_TERM`  | Adverse Event Term reported                      | Headache, Fever, Nausea     |
| `SEVERITY` | Severity of Adverse Event                        | Mild, Moderate, Severe      |
| `TEST`     | Lab Test Name                                    | WBC, Hb, Platelets          |
| `VALUE`    | Numeric Lab Result                               | 91.7, 104.9 ...             |

This structure maps to three CDISC SDTM domains:
- **DM** (Demographics) — `SUBJID`, `AGE`, `SEX`
- **AE** (Adverse Events) — `SUBJID`, `AE_TERM`, `SEVERITY`
- **LB** (Lab Results) — `SUBJID`, `TEST`, `VALUE`

---

## 🎯 Objectives

1. Clean and validate a raw clinical dataset (handle duplicates, invalid values, missing data)
2. Detect outliers in lab results using statistical methods
3. Map cleaned data into CDISC SDTM domain structure (DM, AE, LB)
4. Generate visual summary reports for data insights
5. Export domain-separated data to Excel for downstream use

---

## 🛠️ Tech Stack

| Tool / Library     | Purpose                                       |
|--------------------|-----------------------------------------------|
| **Python 3.x**     | Core programming language                     |
| **Pandas**         | Data loading, cleaning, transformation        |
| **NumPy**          | Numerical operations, statistical computation |
| **Matplotlib**     | Data visualization and report generation      |
| **Jupyter Notebook / Google Colab** | Interactive development environment |
| **MS Excel**       | Final output — SDTM domain export             |

---

## 📁 Project Structure

```
clinical-data-analysis/
│
├── clean_clinical_data.csv      # Raw input dataset (5000+ records)
├── clinical_analysis.ipynb      # Main Jupyter/Colab notebook
├── SDTM_Domains.xlsx            # Output: DM, AE, LB domain sheets
├── clinical_report.png          # Output: Summary visualization report
└── README.md                    # Project documentation (this file)
```

---

## 🚀 How to Run (Google Colab)

### Step 1 — Open Google Colab
Go to [colab.research.google.com](https://colab.research.google.com) and create a new notebook.

### Step 2 — Upload the Dataset
```python
from google.colab import files
uploaded = files.upload()  # Upload clean_clinical_data.csv when prompted
```

### Step 3 — Install & Import Libraries
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import warnings
warnings.filterwarnings('ignore')

df = pd.read_csv('clean_clinical_data.csv')
print(df.shape)
df.head()
```

### Step 4 — Run the Notebook Cells in Order
Follow the sections below to execute each part of the pipeline.

---

## 🔬 Methodology

### 1. Data Loading & Initial Exploration
- Load CSV using `pandas.read_csv()`
- Inspect shape, column types, and first few rows
- Check for null values across all columns

### 2. Data Cleaning & Validation
```python
# Missing value check
print(df.isnull().sum())

# Remove duplicate records
df = df.drop_duplicates()

# Validate AGE (clinical standard: 18–100 years)
invalid_age = df[(df['AGE'] < 18) | (df['AGE'] > 100)]

# Validate categorical fields
print(df['SEX'].unique())        # Expected: ['M', 'F']
print(df['SEVERITY'].unique())   # Expected: ['Mild', 'Moderate', 'Severe']
```

**Validations performed:**
- Age range (18–100 years)
- Sex values (M/F only)
- Severity classification consistency
- Duplicate subject-event-lab combinations

### 3. Outlier Detection (Lab Values)
IQR (Interquartile Range) method applied per test group:

```python
def detect_outliers(group):
    Q1 = group['VALUE'].quantile(0.25)
    Q3 = group['VALUE'].quantile(0.75)
    IQR = Q3 - Q1
    lower, upper = Q1 - 1.5 * IQR, Q3 + 1.5 * IQR
    group['OUTLIER'] = ~group['VALUE'].between(lower, upper)
    return group

df = df.groupby('TEST', group_keys=False).apply(detect_outliers)
```

Each lab test (`WBC`, `Hb`, `Platelets`) gets its own outlier boundary, which is more accurate than applying a single global threshold.

### 4. CDISC SDTM Domain Mapping

**DM Domain (Demographics)**
```python
DM = df[['SUBJID', 'AGE', 'SEX']].drop_duplicates(subset='SUBJID').copy()
DM['DOMAIN'] = 'DM'
DM['STUDYID'] = 'STUDY001'
```

**AE Domain (Adverse Events)**
```python
AE = df[['SUBJID', 'AE_TERM', 'SEVERITY']].drop_duplicates().copy()
AE['DOMAIN'] = 'AE'
AE = AE.rename(columns={'AE_TERM': 'AETERM', 'SEVERITY': 'AESEV'})
```

**LB Domain (Lab Results)**
```python
LB = df[['SUBJID', 'TEST', 'VALUE', 'OUTLIER']].copy()
LB['DOMAIN'] = 'LB'
LB = LB.rename(columns={'TEST': 'LBTESTCD', 'VALUE': 'LBSTRESN'})
```

> ℹ️ **Note:** In real SDTM submissions, additional variables like `USUBJID`, `VISITNUM`, `LBSTRESU`, `AESTDTC` etc. would be required. This project simulates the mapping logic as a self-learning exercise.

### 5. Summary Report Generation
Four plots generated in a 2×2 grid:

| Plot | Description |
|------|-------------|
| Bar Chart | Adverse Event term frequency |
| Pie Chart | Severity breakdown (Mild / Moderate / Severe) |
| Histogram | Age distribution across subjects |
| Box Plot | Lab value distribution per test type (WBC, Hb, Platelets) |

```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
fig.suptitle('Clinical Data Analysis Report', fontsize=16, fontweight='bold')
# ... (see notebook for full code)
plt.savefig('clinical_report.png', dpi=150)
```

### 6. Export to Excel
```python
with pd.ExcelWriter('SDTM_Domains.xlsx') as writer:
    DM.to_excel(writer, sheet_name='DM', index=False)
    AE.to_excel(writer, sheet_name='AE', index=False)
    LB.to_excel(writer, sheet_name='LB', index=False)
```

---

## 📊 Sample Outputs

### Missing Value Summary (Example)
```
SUBJID      0
AGE         0
SEX         0
AE_TERM     0
SEVERITY    0
TEST        0
VALUE       0
dtype: int64
```

### Outlier Count per Test (Example)
```
TEST
Hb          12
Platelets   18
WBC         9
Name: OUTLIER, dtype: int64
```

### SDTM Domain Preview — DM
| STUDYID   | DOMAIN | SUBJID | AGE | SEX |
|-----------|--------|--------|-----|-----|
| STUDY001  | DM     | 8      | 56  | F   |
| STUDY001  | DM     | 10     | 40  | F   |
| STUDY001  | DM     | 11     | 28  | F   |

---

## 📚 CDISC SDTM Reference (Self-Learning)

This project uses simplified versions of the following CDISC SDTM domains:

| Domain | Full Name         | Key Variables Simulated         |
|--------|-------------------|---------------------------------|
| DM     | Demographics      | STUDYID, SUBJID, AGE, SEX       |
| AE     | Adverse Events    | SUBJID, AETERM, AESEV           |
| LB     | Laboratory Tests  | SUBJID, LBTESTCD, LBSTRESN      |

**Resources for further learning:**
- [CDISC SDTM Implementation Guide](https://www.cdisc.org/standards/foundational/sdtm)
- [FDA Data Standards Resources](https://www.fda.gov/industry/study-data-standards-resources)
- CDISC SDTM v1.7 Specification

---

## ⚠️ Limitations & Disclaimer

- This dataset is **mock/simulated** — not real patient data. No PHI (Protected Health Information) is involved.
- The SDTM mapping is a **simplified simulation** for learning purposes. A production SDTM submission requires complete variable lists, controlled terminology, and define.xml.
- Outlier detection uses IQR method — clinical trials may use lab reference ranges instead.
- No statistical inference or efficacy analysis is performed; this is a data management / data quality project.

---

## 🔮 Future Enhancements

- [ ] Add `VISITNUM` and `VISITDY` to simulate longitudinal visit data
- [ ] Implement lab reference range flagging (`LBNRLO`, `LBNRHI`) per SDTM standard
- [ ] Generate a `define.xml`-style data dictionary
- [ ] Add a VS (Vital Signs) domain
- [ ] Integrate with SAS Transport (XPT) format export for FDA-compliant submission simulation

---

## 👤 Author

**Akshay**
B.Pharma Graduate | Aspiring Clinical Data Manager / MCA Candidate
*Durgapur, West Bengal, India*

---

## 📄 License

This project is for academic and personal learning purposes only. The dataset is synthetic and not derived from any real clinical trial.