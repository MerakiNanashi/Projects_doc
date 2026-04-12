# IDEA – AI Call Evaluation Framework

## System Overview

IDEA (Intelligent Data Evaluation Agent) is a large-scale AI system designed for automated evaluation of customer interaction calls under strict constraints of **latency, cost, and evaluation consistency**.

The system was built because traditional manual QA processes and rule-only systems failed to scale and lacked consistency across large volumes of conversational data. Pure LLM solutions were also insufficient due to cost and variability.

This led to a **hybrid architecture combining LLM reasoning, rule-based validation, and ML-based pre-filtering**.

---

## Key Design Goals

- High-throughput evaluation (30K+ calls/day)
- Deterministic and reproducible outputs
- Cost-efficient inference under API constraints
- Modular pipeline with independent evaluation stages
- Hybrid intelligence (rules + ML + LLM reasoning)

---

## System Architecture

Raw Data (WebEngage)  
↓  
Transcription (ASR + diarization + timestamps)  
↓  
Preprocessing (segmentation, filtering)  
↓  
Blank Call Detection (ML Ensemble)  
↓  
Rule-Based Evaluation Layer (deterministic signals)  
↓  
LLM Evaluation (structured reasoning + schema validation)  
↓  
Postprocessing (JSON normalization + flattening)  
↓  
Storage / Reporting Layer  

Each stage was optimized for a specific constraint:  
**latency, noise reduction, cost control, or determinism**

---

## Core System Constraints

- Strict **Pydantic schema enforcement** limited batching flexibility in LLM calls
- Business requirement for **deterministic structured outputs**
- High variability in call quality required **pre-filtering before LLM inference**
- API rate limits enforced **adaptive batching strategies instead of full streaming**

---

## Core Components

### 1. LLM Evaluation Engine (Modular Service Layer)
- Uses Gemini 2.5 Flash, GPT-4o, and local models (Ollama fallback)
- Schema-driven structured prompting with strict JSON contracts
- Pydantic validation ensures deterministic outputs
- Multi-stage evaluation:
  - Discovery
  - Explanation reasoning
  - Actionability scoring

---

### 2. High-Throughput Processing Pipeline
- Async-style batching using ThreadPoolExecutor
- Parallel processing with fault-tolerant job execution
- Retry + fallback routing for API failures
- Sustains 30K+ calls/day throughput under production load

---

### 3. Orchestration Layer (Pluggable Pipeline Architecture)
- Modular per-call execution graph
- Isolated evaluation stages for independent upgrades
- Hybrid integration of rule-based + LLM-based scoring

---

### 4. Rule-Based Validation Layer
- Deterministic keyword and compliance signal detection
- Captures structured business signals (institutional indicators, domain-specific cues)
- Acts as a pre-filter to reduce LLM load and noise

---

### 5. Postprocessing Engine
- Robust JSON repair for malformed LLM outputs
- Multi-response parsing and normalization
- Converts outputs into analytics-ready structured format

---

## Blank Call Detection (ML System)

A pre-LLM filtering system to remove low-signal or invalid calls.

### Features
- Voice Activity Detection (VAD)
- Speech/silence ratio modeling
- Acoustic energy distribution
- MFCC + spectral features
- Segment-based behavioral patterns

### Models
- Balanced Random Forest
- LightGBM
- XGBoost
- Logistic Regression
- LDA

### Ensemble Strategy
Stacked model with Logistic Regression meta-classifier trained on out-of-fold predictions.

---

## Model Benchmarking & Selection Logic

### ASR Models
- Whisper (Small → Large)
- GPT-4o Transcribe
- Gemini 2.5 Flash

### LLM Models
- Gemini 2.5 Flash
- GPT-4o
- GPT-4o Mini
- Mistral 7B

---

### Key Tradeoffs

#### Accuracy vs Latency
- Gemini 2.5 Flash Lite: lower latency but reduced accuracy
- Gemini 2.5: higher accuracy but slower inference
- **Decision:** selected Gemini 2.5 due to accuracy being critical for evaluation correctness

---

#### Cost vs Consistency
- GPT-4o: higher consistency in structured outputs but significantly more expensive
- Gemini 2.5: lower cost with acceptable variance
- **Decision:** defaulted to Gemini 2.5 under strict cost constraints

---

#### Rules vs LLM
- Rules provide deterministic validation
- LLM provides contextual reasoning
- Final system uses a hybrid architecture to balance both

---

#### Batch vs Real-time
- Full batching constrained due to schema validation requirements
- Pydantic-enforced structure required per-request validation
- Result: adaptive micro-batching instead of full batch inference

---

## Reliability Design

- Fallback models for LLM API failures
- Retry mechanisms for transcription and inference failures
- Schema validation prevents malformed outputs from propagating downstream
- Graceful degradation under high load conditions

---

## Failure Cases & Learnings

- Initial batching strategy broke due to strict schema enforcement constraints
- LLM outputs occasionally violated JSON structure under high load
- ASR errors caused downstream evaluation noise
- Learned importance of **pre-filtering before expensive LLM inference**
- Introduced layered validation to stabilize production pipeline

---

## Key Metrics (Prioritized)

- Primary: **Accuracy + Cost efficiency**
- Secondary: Latency + Throughput consistency

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
- Export-ready reporting layer for BI systems

---

## Future Improvements

- Real-time streaming evaluation pipeline
- Active learning loop for classifier improvement
- Improved diarization models for noisy environments
- Vector-based retrieval layer for hybrid evaluation enhancement
