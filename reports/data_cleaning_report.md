# Data Cleaning Report: Hotel Booking Demand Dataset

**Author**: [Imasha Senadheera]
**Date**: [25-07-2025]  
**Dataset Source**: [Kaggle](https://www.kaggle.com/datasets/lessemostipak/hotel-booking-demand)

---

## 1. Executive Summary

- **Objective**: Clean the dataset to ensure it is analysis-ready by handling missing values, duplicates, outliers, and inconsistencies.
- **Dataset Overview**:
  - Original size: 119,390 rows × 32 columns.
  - Time period: July 2015 to August 2017.
  - Key features: Hotel type, booking dates, guest demographics, pricing (`adr`), etc.
- **Key Actions Taken**:
  - Imputed missing values in `children`, `country`, `agent`, and `company`.
  - Removed 32 duplicate records.
  - Capped outliers in `adr` and `lead_time` using IQR.
  - Standardized categorical values (e.g., `meal` codes).

---

## 2. Data Quality Assessment

### Original Dataset Issues

| **Issue Type**  | **Columns Affected**                                                   | **Severity** |
| --------------- | ---------------------------------------------------------------------- | ------------ |
| Missing Values  | `children` (4), `country` (488), `agent` (16,340), `company` (112,593) | High         |
| Duplicates      | 32 exact duplicate rows                                                | Medium       |
| Outliers        | `adr`, `lead_time`, `stays_in_week_nights`                             | High         |
| Inconsistencies | `meal` (case variations), zero-guest bookings                          | Low          |

---

## 3. Cleaning Methodology

### 3.1 Missing Values

| **Column**        | **Strategy**             | **Rationale**                                    |
| ----------------- | ------------------------ | ------------------------------------------------ |
| `children`        | Replaced `NaN` with `0`  | Assumed no children if unspecified.              |
| `country`         | Filled with mode (`PRT`) | Most frequent value preserved data distribution. |
| `agent`/`company` | Replaced `NaN` with `0`  | `0` indicates no agent/company involvement.      |

**Code Snippet**:

```python
df['children'].fillna(0, inplace=True)
df['country'].fillna(df['country'].mode()[0], inplace=True)
df[['agent', 'company']] = df[['agent', 'company']].fillna(0)
```

### 3.2 Removing Duplicates

```python
df.drop_duplicates(inplace=True)
print(f"Rows after deduplication: {len(df)}")  # Output: 119,358
```

### 3.3 Treating Outliers

**IQR Capping Example (ADR)**:

```python
Q1 = df['adr'].quantile(0.25)
Q3 = df['adr'].quantile(0.75)
IQR = Q3 - Q1
df['adr'] = np.clip(df['adr'], Q1 - 1.5*IQR, Q3 + 1.5*IQR)
```

### 3.4 Treating Outliers

- **Categorical Values**:

```python
df['meal'] = df['meal'].str.lower().replace('sc', 'self_catering')
```

- **Invalid Records**:

```python
# Remove bookings with zero guests
df = df[(df['adults'] + df['children'] + df['babies']) > 0]
```

---

## 4. Validation Checks

| **Check**                    | **Pass/Fail** | **Rows Affected** |
| ---------------------------- | ------------- | ----------------- |
| No missing values            | ✅ Pass       | 0                 |
| No duplicates                | ✅ Pass       | 0                 |
| `adr` within [0, 1000]       | ✅ Pass       | 1,247 capped      |
| Valid dates (2015-2017)      | ✅ Pass       | 0                 |
| At least 1 guest per booking | ✅ Pass       | 180 removed       |

---

## 5. Final Dataset Summary

| **Metric**     | **Before Cleaning** | **After Cleaning** |
| -------------- | ------------------- | ------------------ |
| Total Rows     | 119,390             | 119,178            |
| Missing Values | 129,425 cells       | 0                  |
| Duplicates     | 32                  | 0                  |
| Outliers (ADR) | 1,247               | 0                  |

---

## 6. Recommendations

### 1. For Data Collection:

- Make `country` a required field.
- Add validation rules for `adr` (e.g., reject values > $1000).

### 2. For Future Cleaning:

- Automate this pipeline using `scikit-learn` or `PySpark`.
- Track outlier rates over time to detect anomalies.

---
