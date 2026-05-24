# Lead Scoring & Ranking System

## Overview

Production-oriented ML ranking system designed to prioritize high-intent leads under extreme class imbalance and constrained sales bandwidth conditions.

Instead of treating lead conversion as a traditional binary classification problem, the system reformulates the task into a ranking and resource allocation problem optimized for Top-K decision-making.

The project was developed as a production-inspired experimentation and benchmarking framework focused on:
- ranking optimization
- behavioral feature engineering
- imbalance-aware modeling
- explainable lead prioritization
- operational deployment readiness

---

## Problem Statement

Traditional lead scoring systems often fail under:
- severe class imbalance
- sparse behavioral signals
- constrained outreach capacity
- unstable threshold-based classification

In this setting:
- the majority of leads never convert
- sales teams can only contact a limited number of users
- maximizing precision within a constrained Top-K set becomes more valuable than raw classification accuracy

The system therefore prioritizes:

> “Which users should be contacted first to maximize conversion capture under operational constraints?”

rather than:

> “Will this user convert?”

---

## System Goals

- Rank high-intent users under severe imbalance
- Maximize conversion concentration in Top-K leads
- Maintain stable performance across varying ranking thresholds
- Preserve explainability for operational trust
- Prevent temporal leakage in behavioral aggregation
- Support future deployment integration into CRM workflows

---

## High-Level Architecture

Behavioral Events  
↓  
Temporal Aggregation Layer  
↓  
Feature Engineering Pipeline  
↓  
ML Ranking Model (Balanced Random Forest)  
↓  
Lead Scoring  
↓  
Top-K Ranking Engine  
↓  
Operational Prioritization Output

---

## Dataset Characteristics

| Property | Value |
|----------|------|
| Dataset Scale | 100K+ leads |
| Positive Rate | Extremely imbalanced (~sub-1%) |
| Prediction Horizon | ~30 minutes after lead creation |
| Modeling Paradigm | Ranking-first ML system |

---

## Key Design Decisions

### Ranking over Classification

The system intentionally prioritizes ranking quality over raw classification metrics.

Traditional metrics such as:
- accuracy
- ROC-AUC
- thresholded F1

were insufficient for operational decision-making under constrained outreach capacity.

The project instead focused on:
- Precision@K
- Recall@K
- Capture Rate@K
- ranking concentration

---

### Balanced Random Forest Selection

Balanced Random Forest (BRF) was selected after evaluating multiple approaches including:
- Logistic Regression
- XGBoost
- Ensemble systems

BRF demonstrated:
- more stable Top-K performance
- stronger robustness under imbalance
- lower operational complexity
- improved explainability

---

### Explainability-Driven Design

Interpretability was prioritized throughout the system design.

The project intentionally used:
- one-hot encoded behavioral features
- SHAP analysis
- feature importance tracking

to preserve operational transparency and debugging capability.

---

## Feature Engineering Highlights

The system combines:
- behavioral aggregation
- temporal interaction analysis
- hybrid activity mapping
- clustering-assisted feature construction

Behavioral windows were aggregated using:
- short-term intent windows
- multi-session interaction merging
- event-density analysis

Additional engineered signals included:
- activity clustering via KNN-based grouping
- funnel progression indicators
- engagement intensity signals
- session interaction frequency

---

## Evaluation Strategy

The project focused on operationally aligned evaluation.

Primary metrics:
- Precision@K
- Recall@K
- Capture Rate@K

The system was evaluated across:
- multiple Top-K ranges
- varying operational capacities
- ranking stability conditions

---

## Key Insights

- Ranking systems outperform threshold classification under constrained outreach environments
- Stable ranking behavior is more valuable than maximizing raw model complexity
- Behavioral aggregation windows significantly impact signal quality
- Explainability improves operational trust and debugging workflows
- Simpler imbalance-aware models can outperform more complex boosted systems in production-oriented ranking settings

---

## Related Repositories

- [LeadScoringML](https://github.com/MerakiNanashi/LeadScoringML): Demo and extension of the project, exploring lead ranking, uplift modeling, policy benchmarking, and constrained lead allocation using real and semi-synthetic datasets.

---

## Current Scope

This project primarily focuses on:
- experimentation
- benchmarking
- architecture design
- deployment-oriented system validation

rather than full production deployment.

The repository represents a generalized and sanitized reconstruction of production-inspired system design work.

---

## Future Directions

Potential future improvements include:
- Uplift systems
- Learning-to-Rank (LTR) systems
- streaming feature aggregation
- online learning pipelines
- CRM-integrated scoring infrastructure
- adaptive ranking refresh systems
- feature store integration
- drift monitoring systems

---

## Disclaimer

This repository is heavily sanitized and intended for portfolio, educational, and research purposes.

Sensitive business logic, proprietary datasets, deployment infrastructure, and organization-specific implementations have been removed or abstracted.