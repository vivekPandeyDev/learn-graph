# 🚀 LearnGraph

> **An intelligent video learning platform that turns educational videos into searchable knowledge, connected concepts, and personalized learning paths.**

LearnGraph is a backend-focused project designed to demonstrate how a modern distributed system can transform unstructured video content into a structured, searchable knowledge system.

Instead of simply storing and playing videos, LearnGraph understands **what is being taught inside the videos**.

```text
Educational Video
       ↓
   Transcription
       ↓
Concept Extraction
       ↓
 Knowledge Graph
       ↓
 Hybrid Search
       ↓
   Semantic Search
       ↓
      RAG
       ↓
Grounded Answers + Learning Paths
```

📖 **Detailed development plan:** [project-idea.md](./project-idea.md)

---

## 🎯 What Makes LearnGraph Different?

Traditional learning platforms primarily organize content as:

```text
Course → Video → Player
```

LearnGraph organizes it as:

```text
Course
  └── Video
       └── Transcript
            └── Concepts
                 ├── Related Concepts
                 ├── Prerequisites
                 ├── Other Videos
                 └── Learning Paths
```

A learner can search for a concept such as **Kafka Consumer Groups** and immediately discover:

- Which videos explain it
- The exact timestamp where it is discussed
- Related concepts
- Prerequisites
- Other relevant learning material
- An AI-generated answer grounded in the source videos

### The key idea

> **The video is only the raw material. The real product is the knowledge extracted from it.**

---

## 🧠 AI-Powered Knowledge Pipeline

AI is used where it provides meaningful value rather than simply adding a chatbot.

### 1. Speech-to-Text

```text
Video → Audio → Speech-to-Text → Timestamped Transcript
```

The transcript preserves video timestamps so search results can take the learner directly to the relevant moment.

### 2. Concept Extraction

AI identifies important concepts from transcripts:

```text
Kafka
 ├── Producer
 ├── Consumer
 ├── Topic
 ├── Partition
 └── Consumer Group
```

### 3. Relationship Extraction

The system identifies relationships such as:

```text
Consumer
   ↓
READS_FROM
   ↓
Partition
```

```text
Consumer Group
   ↓
ENABLES
   ↓
Parallel Processing
```

### 4. Semantic Search

Embeddings allow users to search by **meaning**, not only exact keywords.

For example:

> "What happens when a Kafka consumer crashes?"

can retrieve content about:

```text
Consumer Failure
Rebalancing
Consumer Groups
Partition Assignment
Offset Management
```

### 5. RAG

The AI assistant does not simply ask an LLM to answer from general knowledge.

Instead:

```text
Question
   ↓
Knowledge Retrieval
   ↓
Concept Search
   ↓
Vector Search
   ↓
Relevant Transcript
   ↓
Context
   ↓
LLM
   ↓
Answer + Video Citations
```

This allows answers to be grounded in the platform's actual educational content.

---

# 🏗️ Technology

| Area | Technology |
|---|---|
| Backend | Java, Spring Boot |
| Architecture | DDD, Event-Driven Architecture |
| Database | PostgreSQL |
| Messaging | Apache Kafka |
| Cache / Rate Limiting | Redis |
| Object Storage | MinIO |
| Media Processing | FFmpeg |
| AI / ML Workers | Python |
| Semantic Search | Embeddings + Vector Search |
| AI | LLM + RAG |
| Frontend | Vue |
| Containers | Docker |
| Orchestration | Kubernetes |
| Observability | OpenTelemetry, Prometheus, Grafana, Loki, Tempo |
| CI/CD | GitHub Actions + GHCR |
| Testing | JUnit, Spring Boot Test, Testcontainers |

---

# ⚙️ Backend Engineering Focus

LearnGraph is intentionally designed around real backend engineering problems.

### Event-driven processing

Large video processing jobs do not block HTTP requests.

```text
API
 ↓
Kafka
 ↓
Workers
 ↓
Processing
```

This allows processing workloads to scale independently.

### Reliability

The system uses patterns such as:

- Outbox Pattern
- Idempotency
- Retry with Backoff
- Dead Letter Queues
- Timeouts
- Circuit Breakers
- Backpressure
- Optimistic Concurrency

### Distributed consistency

The system is designed around the realities of distributed systems:

```text
Duplicate events
Out-of-order events
Worker crashes
Partial failures
Network failures
Database/Kafka inconsistency
Consumer rebalancing
```

The goal is not to assume failures won't happen, but to make the system recover safely.

---

# 🚀 Performance & Scalability

Performance is treated as an engineering problem, not just a technology choice.

### Asynchronous processing

Long-running work is moved away from request threads:

```text
HTTP Request
     ↓
Create Job
     ↓
Kafka
     ↓
Worker Pool
```

### Parallel processing

Independent workloads can be processed concurrently:

```text
             Kafka
               │
       ┌───────┼────────┐
       ↓       ↓        ↓
    Worker  Worker   Worker
       │       │        │
       └───────┼────────┘
               ↓
          Processing
```

Java concurrency techniques, including **virtual threads**, can be evaluated where appropriate.

### Database performance

The system focuses on:

- Proper indexing
- Query optimization
- Pagination
- Batch operations
- Connection-pool tuning
- Optimistic locking
- PostgreSQL full-text search

### Caching

Redis can reduce repeated expensive reads:

```text
Request
  ↓
Redis
  ├── HIT → Response
  └── MISS
       ↓
   PostgreSQL
       ↓
     Redis
```

### Search performance

Search evolves from:

```text
PostgreSQL Full-Text Search
```

to:

```text
Keyword Search
      +
Concept Search
      +
Vector Search
      +
Ranking
```

### Observability-driven optimization

Performance decisions are based on measurements such as:

- P50 / P95 / P99 latency
- Throughput
- Kafka consumer lag
- Database query latency
- Cache hit ratio
- CPU / memory
- Processing duration
- RAG latency

---

# 🔍 Knowledge Graph Without Starting With a Graph Database

The initial knowledge graph is built using PostgreSQL.

```text
Concept
Concept Relationship
Video Concept
Transcript Segment
```

Graph algorithms such as:

- BFS
- DFS
- Shortest Path
- Topological Sort
- Cycle Detection

are used to solve real application problems.

This keeps the initial system simple while allowing the architecture to evolve later if a dedicated graph database becomes justified.

---

# 🎓 Personalized Learning

The knowledge graph can eventually understand:

```text
What the learner knows
        ↓
What the learner wants to learn
        ↓
Missing prerequisites
        ↓
Knowledge graph
        ↓
Learning path
```

Example:

```text
Concurrency
    ↓
Distributed Systems
    ↓
Messaging
    ↓
Kafka
    ↓
Kafka Consumers
    ↓
Consumer Groups
    ↓
Kafka Streams
```

The result is not just **content recommendation**, but a path through the underlying knowledge.

---

# 📈 Observability

Every important workflow can be traced:

```text
Upload
  ↓
Media Service
  ↓
Kafka
  ↓
Processing Worker
  ↓
Transcription
  ↓
Concept Extraction
  ↓
Search
  ↓
RAG
```

Using:

- OpenTelemetry
- Prometheus
- Grafana
- Loki
- Tempo

This makes it possible to investigate both functional failures and performance bottlenecks.

---

# 🧪 Engineering & Testing

The project will include:

- Unit tests
- Integration tests
- Testcontainers
- API tests
- Kafka integration tests
- Database concurrency tests
- End-to-end processing tests
- Failure/recovery tests
- Performance testing

The system will intentionally test failures such as:

```text
Kafka unavailable
Database unavailable
Worker crash
Duplicate event
Processing timeout
Redis unavailable
MinIO unavailable
External AI failure
```

---

# 🗺️ Project Roadmap

The project is developed progressively rather than building the final architecture on day one.

| Phase | Focus |
|---|---|
| 1 | Video Upload & Media Platform |
| 2 | Asynchronous Media Processing |
| 3 | Timestamped Transcription |
| 4 | Concept Extraction & Knowledge Graph |
| 5 | Knowledge & Hybrid Search |
| 6 | Semantic Search & RAG |
| 7 | Personalized Learning Graph |
| 8 | Reliability, Performance & Observability |
| 9 | Docker, Kubernetes & CI/CD |

📖 **Full roadmap:** [project-idea.md](./project-idea.md)

---

# 📚 Project Documentation

The repository will grow with focused documentation:

- [Project Idea & Development Roadmap](./project-idea.md)
- [Architecture](./docs/architecture.md)
- [Architecture Decision Records](./docs/adr/)
- [API Documentation](./docs/api.md)
- [Knowledge Graph](./docs/knowledge-graph.md)
- [Search Architecture](./docs/search.md)
- [RAG Architecture](./docs/rag.md)
- [Event & Kafka Design](./docs/events.md)
- [Performance Engineering](./docs/performance.md)
- [Observability](./docs/observability.md)
- [Deployment](./docs/deployment.md)

> Documentation links will be added as each area is implemented.

---

# 💡 Why This Project?

LearnGraph combines several areas of modern backend engineering into one coherent system:

```text
Java
 +
Spring Boot
 +
DDD
 +
PostgreSQL
 +
Kafka
 +
Redis
 +
Distributed Systems
 +
Concurrency
 +
Graph Algorithms
 +
Search
 +
AI / RAG
 +
Observability
 +
Docker / Kubernetes
```

The important part is that each technology exists to solve a specific problem.

This makes LearnGraph more than a collection of technologies:

> **It is an end-to-end backend system where scalability, asynchronous processing, knowledge modeling, search, AI, reliability, and performance are all connected to a real product problem.**

---

## ⭐ Project Goal

Build a system that can answer:

> **"What does this learning content teach, where is it taught, how is it connected to other knowledge, and what should I learn next?"**

while demonstrating the engineering required to build that system reliably at scale.
