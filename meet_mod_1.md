# AI Meet Moderator

---

## System Overview

AI Meet Moderator is a real-time intelligent moderation and response system designed for live virtual classroom environments. The system classifies incoming chat messages, retrieves relevant knowledge via semantic search (RAG), and generates contextual responses using LLMs while enforcing moderation policies.

It is designed for **low-latency, high-throughput, and context-aware interaction handling in dynamic live environments**.

---

## System Thesis

Traditional classroom moderation systems fail due to:

- rigid rule-based filters that lack contextual understanding
- inability to handle multi-intent or ambiguous student queries
- lack of real-time knowledge grounding

Pure LLM-based systems also fail at scale due to:

- hallucination risk
- high latency
- uncontrolled cost escalation

This system was designed as a **hybrid moderation + retrieval + reasoning architecture**, combining:

> LLM classification + RAG-based retrieval + caching + async execution + policy routing

to achieve **accuracy, latency efficiency, and operational reliability in live classroom environments**.

---

## Key System Enhancements (Critical Production Fixes)

### 1. Message Identity & Mapping Fix (Frontend–Backend Consistency)

A major system issue was incorrect mapping between:
- sender identity (user vs instructor vs system)
- message-response pairing in real-time chat

### Solution Introduced:

- Added `message_id` and `user_id` propagation across full pipeline
- Ensured deterministic mapping between:
  - frontend message event
  - backend processing pipeline
  - returned response injection

### Impact:
- Eliminated response mismatch bugs
- Enabled correct multi-user concurrency handling
- Allowed traceability across full interaction lifecycle

---

### 2. Frontend–Backend Timeout Synchronization (Latency Optimization)

Initial system suffered from:
- desynchronized frontend/backend timeout thresholds
- inconsistent retry behavior
- unnecessary request duplication under slow LLM responses

### Fix:
- Unified timeout contract across frontend + backend
- Standardized retry + sleep intervals in async workers
- Reduced redundant request regeneration

### Impact:
- Improved perceived latency consistency
- Reduced duplicate inference calls
- Stabilized real-time UX under load

---

### 3. Fault-Tolerant Storage Strategy (Redis + DB Hybrid)

Introduced multi-tier storage for resilience and observability.

### Storage Policy:

Only store:

- moderation violations (e.g., policy violations, unsafe content)
- escalation cases (repeat offenders or uncertain classification)
- moderation actions taken
- sampled normal traffic (1–5% for analytics)
- false positives / user appeals

### Storage Architecture:

- Redis → fast transient caching + short-lived state
- PostgreSQL → persistent structured logs + analytics

### Impact:
- Reduced DB load significantly
- Improved observability without full data storage overhead
- Enabled compliance-focused logging strategy

---

### 4. DOM Instability Fix (Frontend Robustness)

Initial system relied on DOM class selectors for message capture in live environments.

### Problem:
- Google Meet / classroom UI updates frequently break DOM structure
- message capture became unreliable over time

### Solution:

- Replaced static DOM selectors with:
  - event-driven observation layer
  - mutation observer-based capture
  - resilient message extraction logic

### Impact:
- Improved frontend stability across UI updates
- Reduced integration breakage risk
- Made system resilient to external UI changes

---

## Key Capabilities

- Real-time chat moderation and response generation
- Multi-class LLM-based intent classification
- Retrieval-Augmented Generation (RAG)
- Async processing pipeline for scalability
- Intelligent caching for latency reduction
- Fault-tolerant distributed backend architecture

---

## System Architecture

The system follows a **layered real-time LLM orchestration architecture**.

---

### 1. Client / Frontend Layer

- Browser-based classroom chat interface
- Event-driven message capture system (mutation observer-based)
- Real-time response rendering
- Maintains message lifecycle state

---

### 2. API Gateway Layer

- FastAPI-based request gateway
- Authentication + rate limiting
- Schema validation for incoming messages
- Ensures safe ingestion of chat traffic

---

### 3. Processing & Routing Layer

Core decision engine of the system.

- LLM-based message classification
- Policy-driven routing engine

Routing outcomes:
- Moderation handling
- RAG-based retrieval path
- Instructor escalation path
- Fallback generic response path

---

### 4. Intelligence Layer (LLM + RAG)

#### Retrieval System (RAG)
- Vector-based semantic search (FAISS-style embedding index)
- Top-K retrieval with relevance filtering
- Domain knowledge grounding

#### Response Generation
- Context-aware LLM reasoning
- Controlled prompting to reduce hallucination
- Structured output enforcement

---

### 5. Optimization Layer

- Redis-based caching (frequent query bypass)
- Response reuse for repeated intents
- Reduces redundant LLM + retrieval calls

---

### 6. Execution Layer

- Async worker queue system
- Background processing for classification + retrieval + generation
- Ensures non-blocking real-time performance

---

### 7. Storage Layer

#### Hybrid Storage System

- Redis → cache + transient state
- PostgreSQL → persistent logs + analytics

### Stored Data Policy:

- Moderation violations (phone numbers, URLs, unsafe content)
- Escalation cases (repeated violations or uncertainty)
- Moderation actions (blocked, flagged, allowed)
- Sampled normal traffic (1–5%)
- False positives / user appeals

---

## Unified Message Flow

1. Message captured via frontend event observer
2. Message sent with `message_id + user_id`
3. API gateway validates request
4. Cache lookup (fast path)
5. LLM classification (intent detection)
6. Routing engine selects execution path:
   - moderation
   - RAG retrieval
   - escalation
7. Vector search (if required)
8. LLM response generation
9. Response stored + cached
10. Response mapped back using message_id
11. Frontend renders correct user-specific response

---

## Key Design Characteristics

- Strong frontend-backend identity consistency via message_id system
- Hybrid reasoning architecture (rules + LLM + retrieval)
- Real-time async execution under latency constraints
- Cache-first optimization strategy
- Fault-tolerant multi-layer storage system
- Resilient frontend integration independent of DOM structure

---

## Key Tradeoffs

### LLM vs Rules
- Rules: fast but rigid
- LLM: flexible but expensive
- Final: hybrid classification system

---

### Retrieval vs Generation
- Retrieval improves factual grounding
- Generation improves contextual reasoning
- Final: RAG-based system for knowledge queries

---

### Latency vs Accuracy
- Cached responses: ~2s
- Full pipeline: 5–10s
- Tradeoff accepted for correctness in live classrooms

---

## Failure Cases & Learnings

- Message-response mismatch due to missing identity propagation → fixed via message_id system
- DOM-based extraction broke frequently due to UI updates → replaced with mutation observer approach
- Initial caching logic caused stale response injection → fixed with TTL-based invalidation
- High concurrency caused LLM rate spikes → mitigated via async worker queue

---

## Performance Metrics

### Latency
- Cached: ~2s
- Full pipeline: 5–10s

### Observations
- Cache significantly reduced LLM load
- Routing layer improved precision of response selection
- Async pipeline stabilized peak traffic behavior

---

## Tech Stack

- Backend: FastAPI, Python (async architecture)
- LLM Layer: Enterprise LLM APIs
- Retrieval: Vector search (FAISS-style embeddings)
- Queue System: Redis-backed worker system
- Cache Layer: Redis
- Database: PostgreSQL (logs + analytics)
- Frontend: Browser-based event-driven integration layer

---

## Summary

AI Meet Moderator is a production-grade real-time LLM orchestration system combining classification, retrieval, and generation into a unified moderation framework.

Its key engineering contributions include:
- identity-safe message-response mapping system
- DOM-resilient frontend integration
- hybrid caching + async architecture
- retrieval-augmented reasoning pipeline
- production-grade fault tolerance design

The system demonstrates how LLM applications in real-time environments require not just model design, but **robust distributed systems engineering, identity tracking, and latency-aware architecture design**.
