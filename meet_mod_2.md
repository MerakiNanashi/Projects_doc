# AI Meet Moderator

---

## 1. Overview

AI Meet Moderator is a real-time intelligent moderation and response system designed for live virtual classroom environments. It automatically classifies incoming chat messages, retrieves relevant context from a domain-specific knowledge base, and generates contextual responses using LLMs while enforcing moderation policies.

The system is designed for:
- low-latency interaction
- high-throughput message processing
- context-aware response generation in live environments

It combines **LLM reasoning, retrieval-augmented generation (RAG), caching, and asynchronous processing** into a unified production system.

---

## 2. Problem Statement

Live classroom chat systems face multiple limitations:

- Rule-based moderation fails on contextual or ambiguous messages
- Pure LLM systems are expensive and inconsistent at scale
- Real-time environments require strict latency guarantees
- External platform UI changes break brittle integrations

The core challenge was to design a system that is:
- accurate in moderation decisions
- fast under high concurrency
- robust to UI and traffic instability
- cost-efficient in production usage

---

## 3. System Thesis

A hybrid architecture was required due to tradeoffs between correctness, cost, and latency.

This system was built as:

> A real-time moderation + retrieval + reasoning pipeline combining LLM classification, semantic search (RAG), caching, and async execution.

Key design goal:
Balance **accuracy, latency, and scalability under real-time classroom constraints**.

---

## 4. System Architecture

The system follows a layered architecture separating frontend ingestion, request handling, intelligence processing, execution, and storage.

---

### 4.1 Frontend Layer (Browser Extension)

A browser extension acts as the client interface inside live classroom platforms.

#### Capabilities

- Captures chat messages in real time from live classroom interfaces
- Can be toggled ON/OFF by the user for controlled activation
- Injects responses directly into the chat UI in real time
- Maintains configurable response templates based on message classification
- Supports dynamic modification of predefined responses for specific intent categories (policy-driven response customization)

#### Output Format:

@SENDERNAME CHATBOTRESPONSE


Key design characteristic:
- Platform-agnostic integration layer
- Independent of backend lifecycle

---

### 4.2 API Gateway Layer

Built using FastAPI.

Responsibilities:
- Authentication and session validation
- Rate limiting per user/session
- Schema validation for message payloads
- Safe ingestion of chat traffic

---

### 4.3 Processing & Routing Layer

This is the decision-making core of the system.

Responsibilities:
- LLM-based message classification
- Policy-driven routing of messages

Routing outcomes:
- Moderation enforcement path
- RAG-based knowledge retrieval path
- Instructor escalation path
- Fallback response path

---

### 4.4 Intelligence Layer (LLM + RAG)

#### Retrieval System (RAG)
- Vector-based semantic search (FAISS-style embeddings)
- Top-K retrieval with ranking and filtering
- Domain knowledge grounding

#### Response Generation
- Context-aware LLM-based response generation
- Controlled prompting to reduce hallucinations
- Structured output formatting for consistency

---

### 4.5 Optimization Layer

Designed to reduce latency and cost.

- Redis caching for frequent queries
- Cache-first execution to bypass LLM calls
- Response reuse for repeated intents
- Significant reduction in redundant inference calls

---

### 4.6 Execution Layer

- Async worker queue system
- Background processing for classification, retrieval, generation
- Non-blocking architecture for high concurrency workloads

---

### 4.7 Storage Layer

Hybrid storage system using Redis + PostgreSQL/MongoDB.

#### Stored Data Policy (Strict Filtering)

Only stored:
- moderation violations (phone numbers, URLs, unsafe content)
- escalation cases (repeat offenders or uncertain classification)
- moderation actions taken
- sampled normal traffic (1–5%)

This ensures compliance-driven logging without full conversation retention.

---

## 5. Unified Message Flow

1. Message captured by browser extension
2. Message sent with:
   - `message_id`
   - `user_id`
3. API gateway validates request
4. Cache lookup is performed
5. LLM classifies message intent
6. Routing engine selects execution path:
   - moderation
   - retrieval (RAG)
   - escalation
7. Vector search retrieves relevant context (if needed)
8. LLM generates response
9. Response is cached and stored
10. Response is mapped back using `message_id`
11. Frontend injects response into chat UI

---

## 6. Key Design Characteristics

- End-to-end identity tracking using `message_id` and `user_id`
- Hybrid reasoning: rules + LLM + retrieval
- Cache-first optimization for latency reduction
- Fully asynchronous execution pipeline
- Frontend extension-based architecture for platform independence
- Strong separation of UI, control, and intelligence layers

---

## 7. Failure Cases & Recovery

This system evolved through multiple real-world engineering challenges.

---

### 7.1 Message Identity Mapping Failure

#### Issue:
Backend lost association between:
- sender identity
- response mapping in frontend UI

#### Cause:
Missing persistent message tracking across system boundaries

#### Resolution:
Introduced:
- `message_id`
- `user_id`
- end-to-end propagation across pipeline

---

### 7.2 Frontend DOM Fragility

#### Issue:
Message extraction broke due to frequent UI updates (e.g., Google Meet DOM changes)

#### Cause:
Dependency on static DOM classes

#### Resolution:
- Replaced DOM selectors with mutation observer-based extraction
- Introduced event-driven message capture layer
- Improved resilience to UI updates

---

### 7.3 Latency Spikes Under Load

#### Issue:
Uncached LLM calls caused latency spikes under high concurrency

#### Resolution:
- Synchronized frontend/backend timeout handling
- Standardized retry and sleep intervals
- Introduced async worker isolation

---

### 7.4 Storage Inefficiency

#### Issue:
Full conversation logging caused unnecessary storage overhead

#### Resolution:
Introduced filtered storage strategy:
- only violations
- escalations
- moderation actions
- sampled normal traffic (1–5%)
- appeals and false positives

---

## 8. Key Tradeoffs

### 8.1 LLM vs Rule-Based Moderation
- Rules: fast but rigid
- LLM: flexible but expensive
- Final: hybrid classification system

---

### 8.2 Retrieval vs Direct Generation
- Retrieval ensures grounding
- Generation ensures flexibility
- Final: RAG for all knowledge-based queries

---

### 8.3 Latency vs Accuracy
- Cached responses: ~2s
- Full pipeline: 5–10s
- Tradeoff accepted for correctness in live environments

---

## 9. Performance Metrics

### Latency
- Cached responses: ~2s
- Full pipeline: 5–10s

### System Observations
- Cache significantly reduced LLM load
- Routing layer improved classification precision
- Async pipeline stabilized peak traffic performance

---

## 10. Optimization Techniques

- Async worker queue system
- Redis-based caching (query + response)
- Message deduplication and normalization
- Semantic similarity optimization
- Controlled LLM prompting for consistency

---

## 11. API Design

- `/query` – message ingestion
- `/process` – async processing
- `/status/{id}` – job tracking
- `/system/health` – service monitoring
- `/system/ready` – dependency checks

---

## 12. Deployment

- FastAPI production backend
- Redis-backed async worker system
- Queue-based distributed processing
- Real-time browser extension integration

---

## 13. Tech Stack

- Backend: FastAPI, Python (async architecture)
- LLM Layer: Enterprise LLM APIs
- Retrieval: Vector search (FAISS-style embeddings)
- Queue System: Redis workers
- Cache Layer: Redis
- Database: PostgreSQL / MongoDB
- Frontend: Browser extension (event-driven injection system)

---

## 14. Summary

AI Meet Moderator is a production-grade real-time LLM system combining classification, retrieval, and generation into a unified moderation pipeline.

It demonstrates:
- robust frontend-backend identity mapping
- resilient browser extension-based integration
- hybrid reasoning + retrieval architecture
- production-grade async and caching system design
- robustness against UI instability and scale pressure
- compliance-aware data storage strategy

---

## 15. Disclaimer

This system represents a generalized and sanitized production implementation. Sensitive prompts, proprietary routing logic, internal datasets, and platform-specific implementations have been abstracted while preserving system design accuracy.
