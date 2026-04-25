#  NASA CMAPSS Engine Degradation Investigation (RUL Prediction)

## 📌 Project Overview
This project focuses on predictive maintenance using the NASA CMAPSS dataset, where the objective is to predict Remaining Useful Life (RUL) of aircraft engines.
Dataset used: CMAPSS FD001 subset

Overview:
* Baseline modeling
* Target engineering (RUL clipping)
* Feature engineering (successful + failed attempts)
* Feature selection
* Model comparison (Random Forest vs XGBoost)

This repository intentionally preserves both successful and failed experiments to demonstrate a real-world machine learning workflow.

---

## 🔄 Pipeline Flow
End-to-end workflow followed:
> Data Loading ➔ EDA ➔ Cleaning ➔ RUL Construction ➔ Baseline Model ➔ RUL Optimization ➔ Feature Engineering ➔ Feature Selection ➔ Model Comparison ➔ Final Model

---

## 📊 Dataset Understanding
* Each engine ➔ `unit_number`
* Each timestep ➔ `time_cycles`
* Each row ➔ sensor readings at a given cycle
* Engines degrade over time until failure

Key concept:
> Each engine has its own lifecycle trajectory

---

## 🔍 Exploratory Data Analysis (EDA)
Performed initial analysis to understand structure and behavior:
* Used `info()` and `describe()` for statistical overview
* Used `nunique()` to count engines
* Used `groupby()` to analyze lifecycle lengths
* Verified sequential cycle progression
* Observed sensor distributions and ranges

**Insight:** Different sensors show varying stability and degradation patterns.

---

## ⚙️ Data Cleaning
**1. Constant / Useless Feature Removal**
Removed:
* `setting_1`, `setting_2`
* `sensor_6` (near-constant)

**Reason:**
> No variance ➔ no predictive power

---

## 🎯 Target Engineering (RUL)
RUL was created as:
> RUL = max(cycle per engine) - current cycle

This converts the dataset into a regression problem.

---

## Key Optimization: RUL Clipping
Tested multiple caps:
> [100, 115, 125, 130, 150]

**Best Result:**
> Cap = 100
> MAE = 8.68
> R² = 83%

**Insight:**
> Early-life engine behavior adds noise ➔ clipping improves learning

---

## Baseline Model
**Model: Random Forest Regressor**
Why:
* Handles non-linear relationships
* Robust to noise
* Minimal preprocessing
* Provides feature importance

### 📊 Baseline Performance
> MAE: ~8.69 flights
> R²: ~82.98%

This served as the reference point for all further experiments.

---

## ⚠️ Experimental Phase (Where Things Broke)

###  Feature Engineering Attempt
Added:
* `_diff` ➔ rate of change
* `_roll_mean` ➔ trend smoothing
* `_roll_std` ➔ instability detection
* `life_stage` ➔ early / mid / late engine life

**Result:**
> MAE ↑ to ~21
> R² ↓ to ~79%

**Insight:**
> More features ≠ better performance. Noise and redundancy degraded model learning.

###  Feature Selection Attempt
Extreme Cleaning:
* Removed `_diff`, `_roll_std`, weak sensors

**Result:**
> Still poor performance (~21 MAE)

###  Goldilocks Approach (Top 20 Features)
Used feature importance to select top features.

**Result:**
> MAE = 19.5
> R² = 82.6%

**Insight:**
> Feature selection helped slightly, but still worse than baseline.

---

##  Key Realization
> Original dataset already contains strong predictive signals.
> Over-engineering degraded performance.

---

## 🏁 Final Strategy
Instead of forcing improvements:
* Reverted to best-performing dataset
* Kept experiments for documentation
* Focused on model comparison

---

##  Model Comparison

**Random Forest (Final Baseline)**
> MAE: 8.68
> R²: 83.03%

**XGBoost (on clean dataset)**
> MAE: 8.46
> R²: 83.27%

**Insight:**
> XGBoost slightly outperformed Random Forest when trained on clean data.

---

##  Key Learnings
1. **Data > Model:** Better data representation matters more than model complexity.
2. **Feature Engineering is Risky:** Bad features hurt more than they help.
3. **Simpler Can Be Better:** Baseline model was already near optimal.
4. **Feature Importance ≠ Feature Elimination Rule:** Low importance features can still contribute to model performance.
5. **Experimentation Matters:** Failures provided more insight than improvements.

---

##  Current Status
* ✅ Baseline model established
* ✅ RUL optimization completed
* ✅ Feature engineering tested (and evaluated critically)
* ✅ Model comparison completed
* ✅ Final model selected

---


## 📝 Final Summary
This project is not just about predicting RUL. It demonstrates:
> how real machine learning works:
> build ➔ break ➔ analyze ➔ refine ➔ conclude

### ⚠️ Pipeline Bug & Feature Engineering Recovery

**The Experiment:**
I introduced time-aware features to improve degradation modeling:
* `_diff` ➔ rate of change (Speedometer)
* `_roll_mean` ➔ trend smoothing (Shock Absorber)
* `_roll_std` ➔ instability detection (Death Rattle)

**The Crash:**
Upon training the Random Forest with these new features, performance inexplicably dropped:
* MAE ↑ to ~21 
* R² ↓ to ~79%
At first, this appeared to be a case of increased feature noise or dimensionality issues.

**The Investigation & Fix:**
Initially, I thought this was the "Curse of Dimensionality" (too much noise degrading the model). However, upon auditing the data pipeline, I discovered a critical logic bug: **The temporary memory drop.** When generating the new features, the pipeline copied the *raw* data (`df_temp = df_train_clean.copy()`), completely overwriting the `Cap = 100` target optimization I had built in the previous phase. 
As a result:
Baseline model used clipped RUL
Feature-engineered model used unclipped RUL

**The Resolution:**
I enforced consistent preprocessing by applying RUL clipping before feature usage:
Ensuring both baseline and feature-engineered models used identical target definitions.

**The Result:**
Once the pipeline was corrected, feature engineering showed strong improvements:
* **MAE:** 6.35 flights
* **R²:**  89.23%

**Key Insight:** Feature engineering is only effective when the target variable and preprocessing pipeline remain consistent across experiments.
