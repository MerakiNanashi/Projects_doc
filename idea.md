# IDEA – AI Call Evaluation Framework

IDEA (Intelligent Data Evaluation Agent) is a production-grade AI system for automated call quality assessment. It processes large-scale call data pipelines and evaluates conversations using a combination of LLM-based reasoning, rule-based validation, and classical machine learning.

The system is designed for high-throughput, low-latency evaluation of customer interaction calls, enabling scalable and consistent QA workflows.

---

## Overview

The pipeline ingests call recordings, generates transcripts, filters low-quality inputs, and evaluates conversations against structured QA frameworks using deterministic LLM outputs.

Key design goals:
- Deterministic and reliable evaluation
- Scalable processing (30k+ calls/day)
- Cost-efficient inference
- Modular and extensible architecture

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
Regex-based Rule Engine  
↓  
LLM Evaluation (Structured Prompting + Schema Validation)  
↓  
Postprocessing (JSON parsing, flattening)  
↓  
Storage / Reporting  


---

## Core Components

### 1. LLM Evaluation Engine
- Built using Gemini 2.5 Flash, OpenAI APIs, and Ollama
- Schema-driven prompting with strict JSON outputs
- Pydantic validation for deterministic and structured responses
- Multi-section evaluation:
  - Discovery
  - Explanation (multi-stage)
  - Actionability

Implementation reference: `llm.py` :contentReference[oaicite:0]{index=0}

---

### 2. High-Throughput Processing Pipeline
- Multi-threaded execution using `ThreadPoolExecutor`
- Async-style batching and parallel processing
- Fault-tolerant design with retries and fallback logic
- Processes 30k+ calls/day with stable throughput

Implementation reference: `main.py` :contentReference[oaicite:1]{index=1}

---

### 3. Orchestration Layer
- Per-call processing abstraction
- Isolated LLM stages for modular evaluation
- Integrated rule-based and LLM-based outputs

Implementation reference: `pipeline.py` :contentReference[oaicite:2]{index=2}

---

### 4. Regex Rule Engine
- Deterministic rule-based validation layer
- Captures domain-specific signals (keywords, compliance indicators)
- Complements LLM outputs for hybrid evaluation

Examples:
- Brand mentions (Amity, LMS, etc.)
- Accreditation signals (UGC, NAAC)
- Financial cues (Loan, EMI)

Implementation reference: `preprocessing.py` :contentReference[oaicite:3]{index=3}

---

### 5. Postprocessing Engine
- Robust parsing of LLM outputs (handles malformed JSON, multiple blocks)
- Flattening into structured tabular format
- Ready for analytics and reporting pipelines

Implementation reference: `postprocessing.py` :contentReference[oaicite:4]{index=4}

---

## Blank Call Detection (ML System)

A dedicated classification system filters out meaningless or low-signal calls before LLM evaluation.

### Feature Engineering

Audio features were extracted using signal processing techniques combined with **Voice Activity Detection (VAD)** to capture speech dynamics and silence patterns.

#### VAD-based Features
- Speech ratio (speech vs silence duration)
- Number of speech segments
- Average and maximum speech segment length
- Silence ratio and longest silence duration
- Speech segment density
- Average pause duration
- Estimated number of speakers (via segment patterns)

#### Acoustic Features
- Peak dB, RMS statistics (mean, std, median, percentiles)
- Energy dynamic range
- Zero Crossing Rate (ZCR mean, std)
- Spectral features:
  - Spectral bandwidth
  - Spectral flatness

#### MFCC Features
- MFCC 1–13 (mean and standard deviation)

These features enabled robust detection of low-signal, silent, or noisy calls before passing data to downstream LLM evaluation.

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

Stacked ensemble with Logistic Regression as meta-learner:

Base Models → Out-of-Fold Predictions → Meta Model (LR)


Final configuration:
- Class-balanced Logistic Regression (liblinear)
- Threshold tuning for optimal F1

---

### Performance

| Metric        | Value |
|--------------|------|
| Accuracy      | 86% |
| Precision     | 89% (positive class) |
| Recall        | 52% (positive class) |
| F1 Score      | 0.65 |

The model prioritizes precision to minimize false positives in downstream evaluation.

---

## Model Benchmarking

Extensive benchmarking was conducted across ASR and LLM models:

### ASR Models
- Whisper Small/Medium/Large
- GPT-4o Transcribe
- Gemini 2.5

### LLM Models
- Gemini 2.5 Flash
- GPT-4o
- GPT-4o Mini
- Mistral 7B

### Evaluation Criteria
- Accuracy (against human-labeled dataset)
- Latency (end-to-end processing time)
- Cost (per-call inference cost)
- Consistency (output variance across runs)

Gemini 2.5 Flash was selected for optimal balance across all dimensions.

---

## Performance Metrics

- 70%+ accuracy per evaluation parameter
- Overall improvement: 64% → 89%
- Manual QA effort reduced by 70%
- Inference cost reduced by 80%
- Latency reduced by 40%
- Throughput: 30k+ calls/day

---

## Design Decisions

### Structured Prompting over Retrieval
- Eliminated retrieval overhead
- Improved determinism and consistency
- Simplified pipeline complexity

### Hybrid Evaluation (LLM + Rules)
- LLM for reasoning and context
- Regex for deterministic compliance checks

### Modular Pipeline
- Each stage independently testable
- Easy model swapping and upgrades

---

## Tech Stack

- Languages: Python
- LLMs: Gemini, OpenAI, Ollama
- ML: scikit-learn, LightGBM, XGBoost
- Data Processing: pandas, NumPy
- Concurrency: ThreadPoolExecutor
- Storage: CSV / Excel (extensible to data warehouse)
- Validation: Pydantic

---

## Outputs

- Structured evaluation reports (CSV / Excel)
- Parameter-level scoring
- Aggregated QA reports
- Ready for BI/dashboard integration

---

## Future Improvements

- Real-time streaming evaluation
- Better diarization models
- Active learning for classifier improvement
- Integration with vector search for hybrid pipelines (if needed)

---

## Disclaimer

This repository represents a reconstructed version of work done in a professional setting. Sensitive data, internal APIs, and proprietary logic have been abstracted or omitted.
