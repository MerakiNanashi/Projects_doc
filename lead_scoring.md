# Lead Scoring & Ranking System

A production-oriented machine learning system for **lead prioritization and conversion prediction** using large-scale behavioral and call interaction data. The system is designed to handle **extreme class imbalance (~0.02% positive class)** and optimize for **high recall at Top-K**, aligning directly with sales team workflows and capacity constraints.

---

## Overview

This project focuses on identifying and ranking high-conversion leads using:

1. **WebEngage Activity Data (Phase 1 – Implemented)**
2. **First Call Activity Data (Phase 2 – Planned Integration)**

The system evolved from a traditional classification setup into a **ranking-based framework**, enabling actionable prioritization of leads under real-world constraints.

---

## Deployment Context

- **Status**: Deployed (internal usage)
- **Prediction Delivery**:
  - Automated **daily Excel-based lead ranking reports**
  - Planned integration with **CRM systems for real-time consumption**

- **Operational Flow**:
  1. Daily lead scoring
  2. Ranking based on conversion likelihood
  3. Top-K selection (aligned with sales capacity)
  4. Distribution to sales teams

---

## Problem Statement

- Identify high-conversion leads from **highly imbalanced datasets**
- Optimize for **Recall@K under fixed sales bandwidth**
- Provide **interpretable and actionable outputs** for business teams

---

## Data Sources

### 1. WebEngage Activity Data (Phase 1)

Pre-call behavioral signals:
- Device, OS, browser
- Traffic source (paid, direct, search)
- Event activity (OTP verification, engagement signals)
- Landing page interactions

---

### 2. First Call Activity Data (Phase 2 – Planned)

Post-call structured signals:
- Convertibility indicators
- Payment intent and behavior
- Objection types and strength
- Conversation dynamics (turn-taking)
- Trust-building signals
- Linguistic features

---

## Feature Engineering

### Data Preparation Strategy

- Designed **hybrid feature mapping** using:
  - **KNN-based clustering**
  - **Manual domain-driven mappings**

- Standardized noisy categorical signals into structured features

---

### Feature Pipeline

A custom pipeline (`LeadFeaturePipeline`) was built to ensure **scalable, consistent, and leakage-safe preprocessing**.

#### Key Capabilities:
- Schema alignment across datasets
- Binary and count-based encoding
- Event aggregation
- Missing value handling
- Time-aware transformations
- Leakage-safe label generation

---

### Temporal Feature Engineering

- Built a **time-aware panel dataset** to capture evolving user behavior
- Aggregated signals across multiple windows:
  - 5 min, 10 min, 15 min, 30 min

#### Key Insight:
- **~70% of user activity occurs within the first 30 minutes**
- Selected **30-minute window** as optimal trade-off between early prediction and signal stability

---

## Modeling Approach

### Class Imbalance Handling

- Extreme imbalance (~0.02% positives) addressed using:
  - **Balanced Random Forest (primary model)**
  - Recall-focused evaluation strategy

---

### Models Experimented

Evaluated a wide range of classification algorithms:
- Logistic Regression
- LDA
- Naive Bayes
- AdaBoost
- Random Forest
- XGBoost

Selected **top-performing models** based on recall-driven metrics and robustness.

Evaluated an ensemble with the top 5 models:
- BRF
- XGBoost
- Perceptron
- Decision Tree
- Random Forest

Selected **BRF** due it performing similar to ensemble, without the complexities.

---

## Ranking-Based System

Transitioned from classification to **ranking-based predictions**:

- Leads scored by conversion probability
- Top-K leads selected for action
- **Evaluated** using:
  - Precision@K
  - Recall@K
  - Capture Rate@K

Across 3 months data: October, December, January. The metrics remained consistent and reliable.

---

## Performance Summary

### Dataset Scale (October)

- **Total leads**: 108,000+
- **Average paid users**: ~40–50

---

### Precision@K (Window Comparison)

| Top-K | 5 min | 10 min | 15 min | 30 min |
|------|------|-------|-------|-------|
| 50   | 0.178 | 0.188 | 0.187 | **0.189** |
| 100  | 0.124 | 0.129 | 0.131 | **0.133** |
| 150  | 0.101 | 0.104 | 0.107 | **0.107** |
| 200  | 0.089 | 0.090 | 0.094 | **0.092** |
| 250  | 0.078 | 0.080 | 0.082 | **0.083** |
| 300  | 0.070 | 0.071 | 0.072 | **0.074** |

---

### Capture Rate@K (Key Metric)

Definition:

> Percentage of total actual converters captured within Top-K predictions

Example:
- Total paid users in month: 42
- Correctly identified in Top-K, K=50: 10

**Capture Rate@50 = 10 / 42 ≈ 24%**

---

### Top-K Precision (30-Min Window)

| Top-K | Precision |
|------|----------|
| 50   | 0.189 |
| 100  | 0.134 |
| 150  | 0.107 |
| 200  | 0.092 |
| 250  | 0.083 |
| 300  | 0.074 |

---

## Impact & Business Outcomes

### Key Outcomes

- **Increased conversion capture rate by ~18%** at fixed sales bandwidth  
  - Reduced time spent on low-intent leads  
  - Improved focus on high-intent users  

- Achieved **high recall at Top-K**, ensuring minimal loss of high-quality leads

- Delivered **automated daily lead ranking reports** for sales teams

---

### Operational Insights

- A significant portion (~33%) of leads remained **uncontacted** during the month  
- Among these, a subset of **high-intent users (predicted positive)** eventually converted  
- Indicates **missed opportunities without prioritization**, reinforcing the need for ranking-based systems

---

## System Architecture

### 1. Data Ingestion
Raw Data (WebEngage)

<div align="center">↓</div>

### 2. Feature Engineering
- Feature Pipeline (Hybrid Mapping + Processing)  
- Temporal Aggregation (Multi-window Panel)

<div align="center">↓</div>

### 3. Model Layer
- Model Training (Balanced Random Forest)  
- Prediction Scoring  

<div align="center">↓</div>

### 4. Ranking Layer
- Top-K Ranking (Lead Prioritization)

<div align="center">↓</div>

### 5. Output Layer
- Daily Excel Reports  

<div align="center">↓</div>

### 6. Future Work
- CRM Integration

---

## Tech Stack

- Python
- Pandas, NumPy
- Scikit-learn
- XGBoost
- SHAP

---

## Key Insights

- **Behavioral signals alone** provide strong early indicators of conversion
- **Temporal aggregation** improves stability and performance
- **Ranking-based systems outperform classification** in real-world sales workflows
- Early user activity is highly predictive of downstream outcomes

---

## Future Work

- Integration of **First Call Activity Data (Phase 2)**
- CRM-based real-time scoring
- Learning-to-rank approaches (LambdaRank, LightGBM Ranker)
- Online learning for dynamic updates

---

## Notes

- Dataset is proprietary and not included
- Feature definitions are representative of production logic
- Metrics are aligned with business evaluation (Top-K prioritization)

---
