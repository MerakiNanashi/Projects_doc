# IDEA – AI Call Evaluation Framework

## System Overview

IDEA (Intelligent Data Evaluation Agent) is a large-scale production AI system for automated call quality evaluation. It combines **LLM-based reasoning, rule-based validation, and classical ML models** to assess customer interactions at scale.

The system was designed to replace manual QA workflows that were:
- non-scalable at high call volumes
- inconsistent across human evaluators
- expensive and slow for operational feedback loops

Pure LLM systems were insufficient due to **cost, variability, and lack of deterministic guarantees**, while rule-only systems lacked contextual reasoning. This led to a **hybrid ML + LLM + rules architecture**.

---

## Key Design Goals

- High-throughput processing (30K+ calls/day)
- Deterministic and structured evaluation outputs
- Cost-efficient inference under API constraints
- Low-latency batch processing system
- Modular and extensible pipeline architecture
- Hybrid intelligence (ML + Rules + LLM reasoning)

---

## System Architecture

Audio Input  
↓  
Transcription (ASR + diarization + timestamps)  
↓  
Preprocessing (segmentation, filtering)  
↓  
Blank Call Detection (ML Ensemble)  
↓  
Rule-Based Validation Layer (deterministic signals)  
↓  
LLM Evaluation (structured prompting + schema validation)  
↓  
Postprocessing (JSON normalization + flattening)  
↓  
Storage / Reporting Layer  

Each stage is independently optimized for constraints such as:
**latency, noise reduction, cost efficiency, or determinism**

---

## Core System Constraints

- Strict schema validation (Pydantic) required deterministic LLM outputs
- API rate limits prevented full-scale naive batching
- Business requirements demanded consistent structured scoring
- High noise in audio required pre-filtering before LLM inference
- Multi-stage pipeline required independent failure isolation

---

## Core Components

### 1. LLM Evaluation Engine (Modular Service Layer)

- Uses Gemini 2.5 Flash, GPT-style APIs, and fallback local models
- Schema-driven prompting ensures structured JSON outputs
- Pydantic validation enforces deterministic response contracts
- Multi-stage evaluation:
  - Discovery
  - Explanation reasoning
  - Actionability scoring

---

### 2. High-Throughput Processing Pipeline

- Parallel execution using ThreadPoolExecutor
- Async-style batching for throughput optimization
- Fault-tolerant retry + fallback routing system
- Handles 30K+ calls/day under production load
- Adaptive batching used due to schema constraints

---

### 3. Orchestration Layer (Pluggable Pipeline Architecture)

- Per-call modular execution graph
- Independent evaluation stages for upgrade flexibility
- Hybrid orchestration of rules + ML + LLM outputs
- Clean separation between preprocessing, inference, and scoring

---

### 4. Rule-Based Validation Layer

- Deterministic keyword + compliance detection engine
- Captures domain-specific signals:
  - institutional indicators
  - business intent signals
  - compliance-related patterns
- Acts as a pre-filter to reduce LLM load and noise amplification

---

### 5. Postprocessing Engine

- Robust JSON recovery for malformed LLM outputs
- Multi-response parsing and normalization
- Converts outputs into structured analytics-ready format
- Enables BI/dashboard integration

---

## Blank Call Detection (ML System)

A dedicated ML filtering system removes low-quality, silent, or non-informative calls before expensive LLM evaluation.

### Purpose

Reduce:
- LLM inference cost
- noise propagation
- false evaluation triggers

---

### Feature Engineering

Audio features extracted using **signal processing + Voice Activity Detection (VAD)**.

#### VAD-Based Features
- Speech-to-silence ratio
- Number of speech segments
- Average / max speech segment length
- Silence ratio and longest silence duration
- Speech segment density
- Average pause duration
- Estimated speaker transitions

#### Acoustic Features
- Peak dB, RMS statistics (mean, std, percentiles)
- Energy distribution and variance
- Zero Crossing Rate (mean, std)

#### Spectral Features
- Spectral bandwidth
- Spectral flatness

#### MFCC Features
- MFCC coefficients (1–13 mean + std)

---

### Models Evaluated

- Balanced Random Forest (BRF)
- LightGBM
- Gradient Boosting Machine (GBM)
- Logistic Regression (LR)
- Linear Discriminant Analysis (LDA)
- XGBoost

---

### Ensemble Strategy

Stacked ensemble architecture:

Base models → Out-of-fold predictions → Logistic Regression meta-model

Final configuration:
- Class-balanced Logistic Regression (liblinear)
- Threshold tuning optimized for F1 score

---

### Performance

| Metric     | Value |
|------------|------|
| Accuracy   | 86%   |
| Precision  | 89%   |
| Recall     | 52%   |
| F1 Score   | 0.65  |

System prioritizes **precision over recall** to avoid false positives in downstream LLM evaluation.

---

## Model Benchmarking & Selection Logic

### ASR Models
- Whisper (Small / Medium / Large)
- GPT-4o Transcribe
- Gemini 2.5

### LLM Models
- Gemini 2.5 Flash
- GPT-4o
- GPT-4o Mini
- Mistral 7B

---

## Key Tradeoffs

### Accuracy vs Latency
- Gemini 2.5 Flash Lite: faster but lower accuracy
- Gemini 2.5: slower but significantly more accurate
- **Decision:** selected Gemini 2.5 due to evaluation correctness being critical

---

### Cost vs Consistency
- GPT-4o: highly consistent structured outputs but expensive
- Gemini 2.5: lower cost with acceptable variance
- **Decision:** defaulted to Gemini 2.5 under strict cost constraints

---

### Rules vs LLM
- Rules ensure deterministic validation
- LLM provides contextual reasoning
- Final system uses hybrid architecture for robustness

---

### Batch vs Real-Time
- Full batching constrained by strict schema validation (Pydantic)
- Required per-request validation for correctness guarantees
- Result: adaptive micro-batching instead of full batch inference

---

## Reliability Design

- Fallback models for LLM API failures
- Retry mechanisms for transcription and inference failures
- Schema validation prevents malformed outputs from propagating
- Graceful degradation under high load conditions

---

## Failure Cases & Learnings

- Initial batching strategy failed due to strict schema enforcement constraints
- LLM outputs occasionally violated JSON structure under high load
- ASR errors introduced noise into downstream evaluation pipeline
- Learned importance of **pre-filtering before expensive inference**
- Introduced layered validation to stabilize production system

---

## Key Metrics (Prioritized)

Primary focus:
- Accuracy
- Cost efficiency

Secondary:
- Latency
- Throughput stability

---

## Performance Impact

- 70%+ parameter-level evaluation accuracy
- System improvement: 64% → 89%
- QA effort reduced by 70%
- Inference cost reduced by 80%
- Latency reduced by 40%
- Scales to 30K+ calls/day

---

## Outputs

- Structured QA evaluation reports (CSV / Excel)
- Parameter-level scoring breakdown
- Aggregated analytics for dashboards
- BI-ready structured datasets

---

## Future Improvements

- Real-time streaming evaluation pipeline
- Active learning loop for ML classifier improvement
- Improved diarization for noisy audio environments
- Vector-based retrieval layer for hybrid evaluation augmentation
