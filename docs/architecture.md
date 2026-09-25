# LearnGraph — Architecture

> **Purpose:** Detailed technical architecture for LearnGraph.  
> **Audience:** Developers, reviewers, and future-you returning to the project after a few months.  
> **Related:** [README](../README.md) · [Project Idea](../project-idea.md)

---

# 1. Architecture Goals

LearnGraph is designed around six primary goals:

1. **Asynchronous processing** — video and AI workloads must not block API requests.
2. **Reliable distributed workflows** — failures, retries, duplicate events, and restarts must be expected.
3. **Knowledge-centric data model** — videos are transformed into concepts and relationships.
4. **Progressive scalability** — introduce infrastructure only when the workload justifies it.
5. **Observable behavior** — every important workflow should be measurable and traceable.
6. **Independent evolution** — media processing, knowledge extraction, search, and learning can evolve independently.

The architecture should optimize for **clarity first, scalability second**.

---

# 2. High-Level Architecture

## Target architecture

```text
                              ┌───────────────────┐
                              │      Vue UI       │
                              └─────────┬─────────┘
                                        │
                                        ▼
                              ┌───────────────────┐
                              │    API Gateway    │
                              │                   │
                              │ Auth / Routing    │
                              │ Rate Limiting     │
                              │ Correlation ID    │
                              └─────────┬─────────┘
                                        │
                  ┌─────────────────────┼──────────────────────┐
                  │                     │                      │
                  ▼                     ▼                      ▼
          ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
          │ Auth / User  │      │ Media Service│      │Learning      │
          │   Service    │      │              │      │Service       │
          └──────────────┘      └──────┬───────┘      └──────┬───────┘
                                       │                     │
                                       ▼                     ▼
                                    PostgreSQL           PostgreSQL
                                       │
                                       ▼
                                     MinIO
                                       │
                                       ▼
                                     Kafka
                                       │
                 ┌─────────────────────┼─────────────────────┐
                 │                     │                     │
                 ▼                     ▼                     ▼
          ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
          │Media Worker  │      │Transcription │      │Knowledge     │
          │              │      │Worker        │      │Worker        │
          │FFmpeg        │      │STT           │      │Concepts      │
          │HLS           │      │Transcript    │      │Relationships │
          └──────────────┘      └──────────────┘      └──────┬───────┘
                                                              │
                                                              ▼
                                                     ┌────────────────┐
                                                     │ Search Service │
                                                     └───────┬────────┘
                                                             │
                                             ┌───────────────┼──────────────┐
                                             ▼               ▼              ▼
                                        PostgreSQL      Vector Store      Redis
                                             │               │
                                             └───────┬───────┘
                                                     ▼
                                               ┌─────────────┐
                                               │ AI / RAG    │
                                               │ Service     │
                                               └─────────────┘
```

This is the **target state**, not the starting point.

---

# 3. Architecture Evolution

The system should evolve in stages.

## Stage 1 — MVP

```text
Vue
 ↓
Media Service
 ↓
PostgreSQL
 ↓
MinIO
```

The goal is simply:

```text
Upload → Store → Play
```

---

## Stage 2 — Asynchronous Processing

```text
Media Service
      ↓
    Kafka
      ↓
Media Worker
      ↓
   FFmpeg
```

Now video processing is asynchronous.

---

## Stage 3 — Knowledge Pipeline

```text
Video
 ↓
Audio
 ↓
STT
 ↓
Transcript
 ↓
Concept Extraction
 ↓
Knowledge Graph
```

---

## Stage 4 — Search

```text
User Query
    ↓
Keyword Search
    ↓
Concept Search
    ↓
Transcript Search
```

---

## Stage 5 — Semantic Search

```text
Query
 ↓
Embedding
 ↓
Vector Search
 ↓
Hybrid Ranking
```

---

## Stage 6 — RAG

```text
Question
 ↓
Retrieval
 ↓
Context
 ↓
LLM
 ↓
Answer + Citations
```

---

## Stage 7 — Production Architecture

Add:

```text
Outbox
Idempotency
Retries
DLQ
Circuit Breakers
Rate Limiting
Caching
Observability
Kubernetes
CI/CD
```

---

# 4. Service Boundaries

Services should be separated according to **business responsibility**, not database tables.

---

## 4.1 API Gateway

### Responsibilities

- Authentication/token validation
- Routing
- Rate limiting
- Correlation ID
- Security headers
- Request filtering
- Centralized request policies

The gateway should remain lightweight.

It should **not** contain business logic.

---

## 4.2 Auth/User Service

Owns:

```text
User
Role
Permission
User Profile
Learning Profile
```

Responsibilities:

- Authentication
- Authorization
- User management
- User learning state

---

## 4.3 Media Service

Owns:

```text
Video
UploadSession
MediaAsset
ProcessingJob
```

Responsibilities:

- Create video
- Upload sessions
- Presigned URLs
- Media metadata
- Video lifecycle
- Processing state
- Media access

Media Service owns the **business state** of a video.

It does not perform CPU-heavy processing itself.

---

## 4.4 Media Worker

Responsibilities:

- Download/read source video
- FFmpeg processing
- HLS generation
- Thumbnail generation
- Audio extraction
- Media metadata extraction

It communicates through Kafka.

Example:

```text
media.video.uploaded
        ↓
Media Worker
        ↓
media.video.processing.completed
```

---

## 4.5 Transcription Worker

Responsibilities:

- Receive audio-processing events
- Speech-to-text
- Timestamp normalization
- Transcript persistence
- Transcript completion events

```text
transcription.requested
        ↓
STT
        ↓
transcription.completed
```

Python is a natural choice here because of the available speech/ML ecosystem.

---

## 4.6 Knowledge Worker

Responsibilities:

- Concept extraction
- Relationship extraction
- Concept normalization
- Confidence calculation
- Knowledge graph updates

```text
transcription.completed
        ↓
Concept Extraction
        ↓
Relationship Extraction
        ↓
Knowledge Graph
```

---

## 4.7 Search Service

Responsibilities:

- Keyword search
- Concept search
- Semantic search
- Hybrid ranking
- Search result aggregation

It should hide search implementation details from clients.

---

## 4.8 Learning Service

Owns:

```text
LearningProgress
UserConcept
LearningPath
LearningPathItem
```

Responsibilities:

- Track learning progress
- Determine missing prerequisites
- Build learning paths
- Track mastered concepts

---

## 4.9 AI / RAG Service

Responsibilities:

- Query processing
- Retrieval orchestration
- Context construction
- LLM calls
- Citation generation
- AI response evaluation

It should not become the source of truth for educational knowledge.

The source of truth remains:

```text
Transcript
Concepts
Relationships
Video metadata
```

---

# 5. Data Ownership

A service should own its business data.

Conceptually:

```text
Auth/User
    ↓
Users

Media
    ↓
Videos
Media Assets
Processing Jobs

Knowledge
    ↓
Concepts
Relationships
Video-Concept mappings

Learning
    ↓
User Knowledge
Learning Progress
Learning Paths
```

Avoid creating a shared database where every service can freely modify every table.

---

# 6. Storage Architecture

LearnGraph uses different storage systems for different data types.

```text
                    LearnGraph Data
                         │
          ┌──────────────┼───────────────┐
          │              │               │
          ▼              ▼               ▼
     PostgreSQL        MinIO           Redis
       │                │                │
       │                │                └─ Cache
       │                │                   Rate Limit
       │                │                   Short-lived State
       │                │
       │                └─ Videos
       │                   Audio
       │                   HLS
       │                   Images
       │
       └─ Metadata
          Transcripts
          Concepts
          Relationships
          Learning State
```

Later:

```text
Vector Store
     ↓
Embeddings
```

---

# 7. PostgreSQL

PostgreSQL is the primary transactional database.

Potential schemas/entities:

```text
video
media_asset
upload_session
processing_job

transcript
transcript_segment

concept
concept_relationship
video_concept
segment_concept

user_concept
learning_progress
learning_path
```

PostgreSQL also provides the initial search implementation.

---

# 8. MinIO

MinIO stores large binary objects.

Example:

```text
media/
├── raw/
│   └── VID-20260925-0001/
│       └── source.mp4
│
├── audio/
│   └── VID-20260925-0001/
│       └── source.wav
│
├── hls/
│   └── VID-20260925-0001/
│       ├── master.m3u8
│       ├── 360p/
│       ├── 480p/
│       └── 720p/
│
└── thumbnails/
    └── VID-20260925-0001/
        └── thumbnail.jpg
```

Large files should not be stored inside PostgreSQL.

---

# 9. Redis

Redis is a supporting system, not the source of truth.

Use cases:

```text
Cache
Rate limiting
Short-lived state
Hot concept data
Search result caching
```

Potential future use:

```text
Distributed locks
```

but only when a real coordination problem exists.

---

# 10. Kafka Architecture

Kafka is the backbone of asynchronous processing.

## Example topics

```text
media.video.uploaded
media.video.processing.requested
media.video.processing.completed
media.video.processing.failed

transcription.requested
transcription.completed
transcription.failed

knowledge.extraction.requested
knowledge.extraction.completed

search.embedding.requested
search.embedding.completed
```

Naming should describe the business event rather than the implementation.

---

# 11. Event Structure

Events should have common metadata.

Example:

```json
{
  "eventId": "evt-123",
  "eventType": "VideoUploaded",
  "aggregateId": "VID-20260925-0001",
  "occurredAt": "2026-09-25T10:00:00Z",
  "correlationId": "req-456",
  "producer": "media-service",
  "schemaVersion": 1,
  "payload": {
    "videoId": "VID-20260925-0001"
  }
}
```

Important fields:

```text
eventId
eventType
aggregateId
occurredAt
correlationId
schemaVersion
payload
```

---

# 12. End-to-End Video Processing Flow

## Step 1 — Create Video

```text
Client
  ↓
POST /videos
  ↓
Media Service
  ↓
PostgreSQL
```

State:

```text
CREATED
```

---

## Step 2 — Generate Upload URL

```text
Client
  ↓
Media Service
  ↓
Presigned URL
```

The client uploads directly to MinIO.

This prevents large files from flowing through the API service.

---

## Step 3 — Upload

```text
Client
   ↓
MinIO
```

---

## Step 4 — Object Event

```text
MinIO
   ↓
ObjectCreated
   ↓
Kafka
```

---

## Step 5 — Processing

```text
Kafka
 ↓
Media Worker
 ↓
FFmpeg
 ├── HLS
 ├── Thumbnail
 └── Audio
```

---

## Step 6 — Transcription

```text
Audio
 ↓
Transcription Worker
 ↓
STT
 ↓
Transcript
```

---

## Step 7 — Knowledge Extraction

```text
Transcript
 ↓
Knowledge Worker
 ↓
Concepts
 ↓
Relationships
```

---

## Step 8 — Search Indexing

```text
Transcript + Concepts
        ↓
   Search Pipeline
        ↓
 ┌──────┴─────────┐
 ▼                ▼
Keyword         Embedding
Index           Vector Store
```

---

# 13. Knowledge Graph Architecture

Initially:

```text
PostgreSQL
```

Data model:

```text
concept
concept_relationship
video_concept
transcript_segment_concept
```

Example:

```text
Kafka
 │
 ├── HAS_CONCEPT → Producer
 ├── HAS_CONCEPT → Consumer
 └── HAS_CONCEPT → Topic

Consumer
 │
 └── READS_FROM → Partition
```

Graph traversal is implemented at the application layer initially.

---

# 14. Graph Algorithms

The knowledge service can provide:

### BFS

```text
Find concepts N relationships away.
```

### DFS

```text
Explore a concept hierarchy.
```

### Shortest path

```text
Find relationship path between concepts.
```

### Topological sort

```text
Build prerequisite order.
```

### Cycle detection

```text
Detect invalid prerequisite relationships.
```

Example learning path:

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
```

---

# 15. Search Architecture

Search evolves in three stages.

## Stage 1 — PostgreSQL Full-Text Search

```text
Query
 ↓
PostgreSQL
 ↓
Transcript
```

Useful for exact terms and basic relevance.

---

## Stage 2 — Concept-Aware Search

```text
Query
 ↓
Concept lookup
 ↓
Related concepts
 ↓
Transcript search
```

Example:

```text
Query: consumer failure
```

May expand to:

```text
Consumer
Consumer Group
Rebalancing
Partition Assignment
Offset
```

---

## Stage 3 — Hybrid Search

```text
                 Query
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Keyword    Concept     Vector
      Search     Search     Search
        │          │          │
        └──────────┼──────────┘
                   ▼
                Ranking
                   ↓
              Final Results
```

The ranking strategy should be measurable and tunable.

---

# 16. Semantic Search

Transcript chunks are converted into embeddings.

```text
Transcript
    ↓
Chunking
    ↓
Embedding Model
    ↓
Vector
    ↓
Vector Store
```

Each vector should retain metadata such as:

```text
videoId
transcriptId
segmentIds
startTime
endTime
conceptIds
```

This allows vector results to become actual video citations.

---

# 17. RAG Architecture

RAG should be a pipeline rather than a single LLM call.

```text
User Question
      ↓
Query Processing
      ↓
Candidate Retrieval
      ├── Keyword
      ├── Concepts
      └── Vector Search
              ↓
           Ranking
              ↓
        Context Builder
              ↓
             LLM
              ↓
       Answer Generation
              ↓
        Citation Mapping
              ↓
      Answer + Timestamps
```

The final response should ideally identify:

```text
Course
Video
Timestamp
Transcript segment
Concept
```

---

# 18. AI Responsibility Boundaries

AI should assist the system rather than own the entire system.

AI is appropriate for:

```text
Speech recognition
Concept extraction
Relationship extraction
Embeddings
Semantic similarity
Answer generation
Learning explanation
```

Traditional backend logic remains responsible for:

```text
Authentication
Authorization
Transactions
State transitions
Data integrity
Idempotency
Graph traversal
Scheduling
Retry
Persistence
```

This separation makes the system easier to reason about and test.

---

# 19. Reliability Architecture

Distributed processing assumes failures.

Important patterns:

```text
Outbox
Idempotency
Retry
Dead Letter Queue
Timeout
Circuit Breaker
Backpressure
```

---

# 20. Outbox Flow

Problem:

```text
DB transaction succeeds
Kafka publish fails
```

Without an outbox:

```text
Database = SUCCESS
Kafka = FAILURE
```

With outbox:

```text
Transaction
 ├── Business Data
 └── Outbox Event
          ↓
      Publisher
          ↓
        Kafka
```

The outbox is part of the same database transaction.

---

# 21. Idempotent Consumer

Kafka delivery can result in duplicate processing.

Example:

```text
Event: VideoUploaded
Event ID: evt-123
```

Consumer receives:

```text
evt-123
evt-123
```

The second processing should not corrupt state.

Possible strategies:

```text
Processed event table
+
Unique event ID constraint
```

or domain-specific idempotency.

---

# 22. Retry and DLQ

Transient failure:

```text
Worker
 ↓
STT Service
 ↓
Timeout
```

Retry:

```text
Attempt 1
 ↓
Backoff
 ↓
Attempt 2
 ↓
Backoff
 ↓
Attempt 3
 ↓
DLQ
```

Retries should have:

- Maximum attempts
- Exponential backoff
- Jitter where appropriate
- Error classification

Do not retry permanent failures forever.

---

# 23. Concurrency Architecture

Potential concurrent workloads:

```text
Multiple videos
Multiple transcript chunks
Multiple embedding requests
Multiple Kafka partitions
Multiple search operations
```

Possible Java mechanisms:

```text
ExecutorService
CompletableFuture
Virtual Threads
Kafka consumer concurrency
Batch processing
```

Concurrency should be bounded by downstream resources.

For example:

```text
1000 virtual threads
        ↓
PostgreSQL
        ↓
Connection pool = 20
```

does not mean 1000 database operations can run simultaneously.

The database connection pool becomes the limiting resource.

---

# 24. Data Integrity

Important concurrent scenarios:

```text
Two workers update same processing job
Two consumers process same event
Two requests create same logical resource
Two workers update concept relationships
```

Use:

```text
Unique constraints
Optimistic locking
Atomic updates
Transactions
Idempotency
Database constraints
```

Database constraints should enforce critical invariants whenever possible.

---

# 25. Performance Architecture

Performance optimization follows:

```text
Measure
 ↓
Identify bottleneck
 ↓
Change
 ↓
Measure again
```

Important metrics:

```text
P50 latency
P95 latency
P99 latency
Throughput
CPU
Memory
Database latency
Connection pool usage
Kafka lag
Redis hit ratio
Search latency
RAG latency
```

---

# 26. Performance Strategies

## API

- Pagination
- Request validation
- Compression where useful
- Efficient serialization
- Connection pooling

## Database

- Correct indexes
- Query optimization
- Batch operations
- Avoid N+1 queries
- Proper transaction boundaries
- Optimistic locking
- Connection pool tuning

## Kafka

- Appropriate partition count
- Batch processing
- Consumer concurrency
- Consumer lag monitoring

## Redis

- Cache-aside
- TTL
- Cache invalidation
- Avoid excessive cache size

## Search

- Indexing
- Candidate filtering
- Hybrid retrieval
- Result limits
- Ranking optimization

## AI

- Chunking strategy
- Batch embeddings
- Cache embeddings
- Limit context size
- Avoid unnecessary LLM calls

---

# 27. Backpressure

A critical concern:

```text
Upload Rate
     ↓
100 videos/min
     ↓
Transcription Capacity
     ↓
20 videos/min
```

The system must not blindly create unlimited concurrent processing.

Kafka naturally provides buffering.

Workers should process at a sustainable rate.

Monitor:

```text
Consumer lag
Processing duration
Worker utilization
External API limits
Database load
```

---

# 28. Caching Strategy

Use cache-aside:

```text
Request
  ↓
Redis
  │
  ├── HIT → Return
  │
  └── MISS
       ↓
    PostgreSQL
       ↓
     Redis
       ↓
    Return
```

Potential cache candidates:

```text
Popular concepts
Popular videos
Frequently requested learning paths
Search results
Video metadata
```

Avoid caching data where stale results could violate critical business rules.

---

# 29. Observability Architecture

Use:

```text
OpenTelemetry
      │
 ┌────┼────┐
 ▼    ▼    ▼
Trace Metrics Logs
 │      │     │
 ▼      ▼     ▼
Tempo Prometheus Loki
             │
             ▼
           Grafana
```

---

# 30. Distributed Trace Example

A user uploads a video.

Trace:

```text
HTTP POST /videos
       │
       ▼
Media Service
       │
       ▼
PostgreSQL
       │
       ▼
Kafka
       │
       ▼
Media Worker
       │
       ├── MinIO
       ├── FFmpeg
       └── PostgreSQL
              │
              ▼
          Kafka Event
              │
              ▼
      Transcription Worker
              │
              ▼
             STT
```

Correlation and trace identifiers should allow the workflow to be investigated end-to-end.

---

# 31. Security Architecture

Security responsibilities:

```text
Gateway
 ├── JWT validation
 ├── Rate limiting
 └── Request filtering

Services
 ├── Authorization
 ├── Business-level access control
 └── Input validation

MinIO
 └── Presigned/signed access
```

JWT validation should include appropriate checks for:

```text
iss
aud
exp
nbf
signature
```

Never trust client-supplied identity headers.

---

# 32. Deployment Architecture

Development:

```text
Docker Compose
```

Production-style deployment:

```text
GitHub
   ↓
GitHub Actions
   ↓
Build + Test
   ↓
Docker Image
   ↓
GHCR
   ↓
Kubernetes
```

Kubernetes eventually manages:

```text
API services
Workers
Kafka consumers
Search services
AI services
```

Stateful infrastructure should be introduced carefully rather than assuming Kubernetes automatically solves persistence.

---

# 33. Scaling Strategy

Different workloads scale independently.

Example:

```text
API traffic
    ↓
Scale API instances

Video processing
    ↓
Scale Media Workers

Transcription
    ↓
Scale Transcription Workers

Concept extraction
    ↓
Scale Knowledge Workers

Search
    ↓
Scale Search Service

RAG
    ↓
Scale AI Service
```

Kafka provides buffering between many of these components.

---

# 34. Example Scaling Scenario

Suppose:

```text
100 videos uploaded
```

Media processing becomes the bottleneck.

Instead of increasing API servers:

```text
API = 2 instances

Media Workers = 10 instances
```

Kafka absorbs the workload.

Later, transcription may become the bottleneck:

```text
Media Workers = 10
Transcription Workers = 20
```

Each pipeline stage can therefore scale according to its own workload.

---

# 35. Failure Scenarios

The system should explicitly test:

```text
Kafka unavailable
PostgreSQL unavailable
Redis unavailable
MinIO unavailable
Worker crashes
Duplicate event
Out-of-order event
Slow STT provider
STT timeout
LLM timeout
Database connection exhaustion
Kafka consumer rebalance
Partial processing
Application restart
```

For every failure ask:

```text
What state remains?
Can processing resume?
Will the event be duplicated?
Can the operation be retried?
Will data become inconsistent?
How does the user recover?
```

---

# 36. Testing Architecture

## Unit tests

```text
Domain rules
State transitions
Graph algorithms
Ranking
Chunking
```

## Integration tests

Use Testcontainers for:

```text
PostgreSQL
Kafka
Redis
MinIO
```

## End-to-end tests

Example:

```text
Upload
 ↓
Object Created
 ↓
Kafka
 ↓
Processing
 ↓
Transcript
 ↓
Concepts
 ↓
Search
```

## Failure tests

Simulate:

```text
Worker crash
Database outage
Kafka outage
Duplicate event
Timeout
```

---

# 37. Architecture Decisions

Important decisions should be documented in `docs/adr/`.

Examples:

```text
ADR-001 PostgreSQL as initial knowledge graph
ADR-002 Kafka for asynchronous processing
ADR-003 MinIO for object storage
ADR-004 Outbox for reliable event publication
ADR-005 Python workers for AI/ML processing
ADR-006 Hybrid search architecture
ADR-007 Service boundaries
```

Each ADR should contain:

```text
Context
Decision
Alternatives
Trade-offs
Consequences
```

---

# 38. What the Architecture Deliberately Avoids

Initially avoid:

```text
❌ Microservice per entity
❌ Graph database
❌ Elasticsearch
❌ Kubernetes-first development
❌ Complex agent architecture
❌ Distributed transactions everywhere
❌ Redis as primary database
❌ LLM controlling business state
```

Each can be introduced later if there is a demonstrated need.

---

# 39. Evolution to a Graph Database

The initial architecture uses:

```text
PostgreSQL
+
Application-level graph algorithms
```

If the graph eventually becomes very large or traversal patterns become difficult to support efficiently, evaluate a graph database.

The decision should be based on:

```text
Graph size
Traversal complexity
Query latency
Operational cost
Development complexity
```

Do not introduce a graph database simply because the project contains a graph.

---

# 40. Architecture Principles

### Principle 1 — Async by default for long-running work

```text
HTTP ≠ Video Processing
HTTP ≠ Transcription
HTTP ≠ Embedding Generation
```

### Principle 2 — Database is the source of truth

Redis and search indexes are derived/supporting systems.

### Principle 3 — Events must be replay-safe

Consumers should tolerate duplicate delivery.

### Principle 4 — Scale independently

Workers should scale separately from APIs.

### Principle 5 — Measure before optimizing

No performance claim without measurements.

### Principle 6 — AI does not own business truth

AI extracts and generates; backend systems validate and persist.

### Principle 7 — Complexity must earn its place

Every infrastructure component should solve a demonstrated problem.

---

# 41. End-State Request Flow

## Search

```text
User
 ↓
Gateway
 ↓
Search Service
 ├── PostgreSQL
 ├── Concept Graph
 ├── Vector Store
 └── Redis
 ↓
Ranked Results
 ↓
Video + Timestamp
```

## RAG

```text
User
 ↓
Gateway
 ↓
AI/RAG Service
 ↓
Search Service
 ↓
Knowledge Graph
 ↓
Vector Search
 ↓
Context Builder
 ↓
LLM
 ↓
Citation Mapper
 ↓
Answer
```

## Upload

```text
User
 ↓
Gateway
 ↓
Media Service
 ↓
PostgreSQL
 ↓
Presigned URL
 ↓
MinIO
 ↓
Object Event
 ↓
Kafka
 ↓
Media Worker
 ↓
Transcription
 ↓
Knowledge Extraction
 ↓
Search Indexing
```

---

# 42. Final Architecture

The complete system can be viewed as four major layers:

```text
┌─────────────────────────────────────────────┐
│                 EXPERIENCE                  │
│             Vue + API Gateway              │
├─────────────────────────────────────────────┤
│                  DOMAIN                    │
│   Media | Knowledge | Search | Learning    │
├─────────────────────────────────────────────┤
│                PROCESSING                  │
│ Kafka | Workers | FFmpeg | AI | RAG       │
├─────────────────────────────────────────────┤
│               INFRASTRUCTURE               │
│ PostgreSQL | Redis | MinIO | Observability│
│ Docker | Kubernetes | CI/CD                │
└─────────────────────────────────────────────┘
```

The architecture is intentionally evolutionary:

```text
Simple
  ↓
Asynchronous
  ↓
Distributed
  ↓
Searchable
  ↓
Knowledge-aware
  ↓
AI-assisted
  ↓
Observable
  ↓
Scalable
```

The goal is not to build the most complicated architecture possible.

The goal is to build a system where **each architectural decision has a concrete reason, measurable benefit, and understood trade-off**.

---

# 43. Related Documentation

- [README](../readme.md)
- [Project Idea & 3–4 Month Roadmap](../project-idea.md)
- [API Design](./api.md)
- [Knowledge Graph](./knowledge-graph.md)
- [Search Architecture](./search.md)
- [RAG Architecture](./rag.md)
- [Event & Kafka Design](./events.md)
- [Performance Engineering](./performance.md)
- [Observability](./observability.md)
- [Deployment](./deployment.md)
- [Architecture Decision Records](./adr/)

> These documents are intentionally created progressively as the corresponding part of the system is implemented.
