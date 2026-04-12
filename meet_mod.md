# AI Meet Moderator

## Overview

AI Meet Moderator is a real-time intelligent moderation and response system designed for live virtual classroom environments. The system automatically classifies incoming chat messages, retrieves relevant information from a domain-specific knowledge base, and generates contextual responses while enforcing moderation policies.

It leverages large language models (LLMs), semantic retrieval, and asynchronous processing to deliver scalable, low-latency responses in live settings.

---

## Key Capabilities

- Real-time chat moderation and automated response generation
- Multi-class message classification using LLMs
- Retrieval-Augmented Generation (RAG) with semantic search
- Asynchronous job processing for scalability
- Intelligent caching for latency optimization
- Fault-tolerant API architecture for production environments

---

## System Architecture

### Backend

- REST API built using FastAPI
- Middleware-based rate limiting and request control
- CORS-enabled for cross-platform integrations
- Modular architecture with controllers, services, and utilities

### Message Processing Pipeline

1. Incoming message received via API or client integration
2. Cache lookup for previously processed queries
3. LLM-based classification into predefined categories
4. Conditional routing:
   - Moderation responses (policy violations)
   - Deferred responses (instructor-handled queries)
   - Knowledge base retrieval pipeline
5. Semantic retrieval using vector search
6. Context-aware response generation using LLM
7. Response caching and persistence

---

## Core Components

### 1. Message Classification

LLM-driven classification system that categorizes messages into structured intent groups such as:

- Moderation-related messages
- Knowledge base queries
- Instructor-directed questions
- Unsupported or out-of-scope queries

The system enforces structured outputs and validation to ensure consistency and reliability.

---

### 2. Retrieval-Augmented Generation (RAG)

- Embedding-based semantic search over a domain-specific knowledge base
- Vector index for efficient similarity retrieval
- Top-K document retrieval with relevance filtering
- Context construction from multiple sources for grounded responses

---

### 3. Response Generation

- Context-aware LLM responses grounded in retrieved knowledge
- Controlled prompting to minimize hallucination
- Timeout handling with graceful degradation

---

### 4. Asynchronous Processing

- Background job queue for non-blocking request handling
- Worker-based execution model for scalability
- Job lifecycle tracking with status polling

---

### 5. Caching Layer

- In-memory caching for repeated queries
- Reduces response latency significantly for frequent requests
- Applied across retrieval and response stages

---

### 6. Client Integration Layer

- Real-time message ingestion via browser-based client integration
- DOM observation and event-driven message capture
- Deduplication and response injection mechanisms

---

## Data Storage

### Primary Storage

- **MongoDB / PostgreSQL**
  - Message logs
  - User and session identifiers
  - Query-response mappings
  - Classification metadata
  - System-level analytics

### Data Model (Abstracted)

**Messages**
- `message_id`
- `user_id`
- `session_id`
- `message_text`
- `normalized_text`
- `timestamp`

**Responses**
- `response_id`
- `message_id`
- `classification`
- `response_text`
- `latency_ms`

---

## Authentication & Security

- API key-based authentication for service access
- Rate limiting per client to prevent abuse
- Input validation and schema enforcement
- Policy-driven moderation layer to ensure safe interactions

---

## Fault Tolerance & Fallbacks

- Graceful handling of model timeouts and failures
- Default fallbacks for:
  - Low-confidence retrieval results
  - Missing knowledge base matches
  - Classification uncertainties
- Safe degradation to generic responses when required

---

## Performance Metrics

- Evaluated on a diverse dataset of 1,000+ queries
- High response accuracy across mixed query types (~90%)
- Latency:
  - Cached responses: ~2 seconds
  - Uncached responses: ~5–10 seconds

---

## Optimization Techniques

- Asynchronous execution with worker queues
- Response and retrieval caching
- Semantic similarity calibration
- Message deduplication and normalization
- Controlled generation parameters for consistent outputs

---

## API Design

### Core Endpoints (Sanitized)

- `/query` – Handles incoming user queries
- `/process` – Asynchronous message processing endpoint
- `/status/{id}` – Processing status tracking
- `/system/health` – Service health check
- `/system/ready` – Dependency readiness validation

---

## Deployment

- Production-ready FastAPI service
- Worker-based scalable architecture
- Queue-backed processing system
- Designed for high-throughput, real-time environments

---

## Tech Stack

- **Backend:** FastAPI, Python
- **LLM Integration:** Enterprise LLM APIs
- **Vector Search:** FAISS (or equivalent)
- **Queue System:** Redis-backed job queue
- **Caching:** Redis
- **Database:** MongoDB / PostgreSQL
- **Client Integration:** Browser-based extension layer

---

## Summary

AI Meet Moderator is a production-grade intelligent moderation system that combines LLM reasoning, semantic retrieval, and scalable backend engineering to automate real-time classroom interactions. It is designed to operate reliably under high throughput while maintaining accuracy, latency efficiency, and moderation integrity.

---

## Disclaimer

This repository presents a generalized and sanitized version of the system for demonstration purposes. Sensitive implementation details, proprietary logic, internal prompts, endpoint structures, and domain-specific data have been abstracted or modified to ensure confidentiality while preserving the overall system design and technical concepts.