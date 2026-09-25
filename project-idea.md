# LearnGraph --- Intelligent Video Knowledge Platform

> **Project type:** Backend-heavy learning platform\
> **Primary stack:** Java, Spring Boot, PostgreSQL, Kafka, Redis, MinIO,
> Docker\
> **AI stack:** Python workers, speech-to-text, embeddings, RAG, LLM\
> **Frontend:** Vue\
> **Target development time:** 3--4 months\
> **Main goal:** Build one serious backend system that demonstrates
> modern backend engineering rather than another CRUD application.

------------------------------------------------------------------------

# 1. Project Vision

LearnGraph is an educational video platform where the important
knowledge inside videos becomes **searchable, connected, and usable for
learning**.

A normal video platform looks like:

``` text
User
  ↓
Upload Video
  ↓
Store Video
  ↓
Play Video
```

LearnGraph evolves this into:

``` text
Video
  ↓
Transcript
  ↓
Concepts
  ↓
Relationships
  ↓
Knowledge Graph
  ↓
Search
  ↓
Semantic Search
  ↓
RAG
  ↓
Personalized Learning Path
```

The core idea is:

> **Every important concept in a video should become a searchable
> knowledge unit that can be connected to other concepts.**

For example, a Kafka course may contain:

``` text
Kafka
 ├── Producer
 ├── Consumer
 ├── Topic
 ├── Partition
 ├── Consumer Group
 ├── Offset
 └── Kafka Streams
```

And relationships:

``` text
Kafka
 ├── HAS_CONCEPT → Producer
 ├── HAS_CONCEPT → Consumer
 ├── HAS_CONCEPT → Topic
 └── HAS_CONCEPT → Partition

Consumer Group
 └── USES → Consumer

Consumer
 └── READS_FROM → Partition
```

The user can then search for a concept and jump directly to the relevant
timestamp in a video.

------------------------------------------------------------------------

# 2. Why This Project Exists

The purpose is not simply to build a video application.

The project should demonstrate practical experience with:

-   Domain-driven design
-   Event-driven architecture
-   Asynchronous processing
-   Distributed systems
-   Concurrent processing
-   Data consistency
-   PostgreSQL modeling and indexing
-   Redis caching
-   Kafka
-   Object storage
-   Media processing
-   Search
-   Graph algorithms
-   Vector search
-   RAG
-   AI-assisted processing
-   Reliability patterns
-   Observability
-   Containerization
-   CI/CD
-   Kubernetes

Instead of creating many unrelated small projects, LearnGraph becomes
one long-running project where complexity is introduced gradually.

------------------------------------------------------------------------

# 3. End-State Architecture

The final architecture should approximately look like this:

``` text
                         ┌──────────────────┐
                         │     Vue UI       │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │   API Gateway    │
                         └────────┬─────────┘
                                  │
             ┌────────────────────┼────────────────────┐
             │                    │                    │
             ▼                    ▼                    ▼
      ┌─────────────┐      ┌─────────────┐      ┌──────────────┐
      │ Auth/User   │      │    Media    │      │   Learning   │
      │  Service    │      │   Service   │      │   Service    │
      └─────────────┘      └──────┬──────┘      └──────┬───────┘
                                  │                    │
                                  ▼                    ▼
                               MinIO              PostgreSQL
                                  │
                                  ▼
                                Kafka
                                  │
             ┌────────────────────┼────────────────────┐
             │                    │                    │
             ▼                    ▼                    ▼
       Transcription        Concept Worker      Media Worker
          Worker                │                    │
             │                  │                    │
             ▼                  ▼                    ▼
       Transcript         Knowledge Graph       HLS/Thumbnail
                                  │
                                  ▼
                           Search Service
                              │       │
                              ▼       ▼
                         PostgreSQL  Vector Store
                              │       │
                              └───┬───┘
                                  ▼
                             AI / RAG
                              Service
```

**Important:** this is the final target, not the starting architecture.

The system should begin as a small number of services and evolve as
requirements justify additional components.

------------------------------------------------------------------------

# 4. Development Philosophy

Do not start by building 10 microservices.

Build in this order:

``` text
Working MVP
     ↓
Reliable processing
     ↓
Event-driven processing
     ↓
Search
     ↓
Knowledge graph
     ↓
Semantic search
     ↓
RAG
     ↓
Concurrency & scalability
     ↓
Observability
     ↓
Production deployment
```

Every phase should produce something that actually works.

------------------------------------------------------------------------

# 5. Core User Journey

A complete user journey eventually becomes:

``` text
1. User uploads video
        ↓
2. Video stored in MinIO
        ↓
3. Upload event generated
        ↓
4. Processing pipeline starts
        ↓
5. Video converted / HLS generated
        ↓
6. Audio extracted
        ↓
7. Speech converted to text
        ↓
8. Timestamped transcript stored
        ↓
9. Important concepts extracted
        ↓
10. Concept relationships identified
        ↓
11. Knowledge graph updated
        ↓
12. Transcript chunks embedded
        ↓
13. Search index/vector store updated
        ↓
14. User searches a topic
        ↓
15. Relevant concepts and transcript segments found
        ↓
16. User jumps to exact video timestamp
        ↓
17. User can ask a question
        ↓
18. RAG retrieves relevant knowledge
        ↓
19. LLM generates grounded answer
        ↓
20. Answer contains video/timestamp citations
```

------------------------------------------------------------------------

# 6. Phase 1 --- Media Platform

## Goal

Build a reliable video upload and playback backend.

### Features

-   Video metadata
-   Upload session
-   Presigned upload URL
-   MinIO storage
-   Video status
-   HLS generation
-   Thumbnail generation
-   Video processing state
-   Kafka event
-   Outbox pattern

### Initial flow

``` text
Client
  ↓
Media API
  ↓
Create Video
  ↓
Generate Presigned PUT URL
  ↓
Client uploads directly to MinIO
  ↓
MinIO ObjectCreated event
  ↓
Kafka
  ↓
Processing Worker
```

### Suggested video lifecycle

``` text
CREATED
   ↓
UPLOADING
   ↓
UPLOADED
   ↓
PROCESSING
   ↓
READY
```

Failure state:

``` text
PROCESSING
   ↓
FAILED
```

### Important domain concepts

``` text
Video
UploadSession
MediaAsset
ProcessingJob
```

### Media assets

``` text
RAW_VIDEO
HLS_MASTER
VIDEO_360P
VIDEO_480P
VIDEO_720P
VIDEO_1080P
THUMBNAIL
PREVIEW_IMAGE
```

### Example object layout

``` text
media/
  raw/
    VID-20260925-0001/
      source.mp4

  hls/
    VID-20260925-0001/
      master.m3u8
      720p/
      480p/
      360p/

  thumbnails/
    VID-20260925-0001/
      thumbnail.jpg
```

### Backend concepts learned

-   Presigned URLs
-   Object storage
-   Event-driven processing
-   Kafka producers/consumers
-   Outbox
-   Idempotency
-   Transaction boundaries
-   DDD aggregate lifecycle

------------------------------------------------------------------------

# 7. Phase 2 --- Video Processing Pipeline

## Goal

Process uploaded videos asynchronously.

``` text
Video Uploaded
      ↓
Kafka
      ↓
Processing Worker
      ├── FFmpeg
      ├── HLS generation
      ├── Thumbnail
      └── Audio extraction
```

### FFmpeg responsibilities

Initially:

``` text
MP4
 ↓
Audio WAV
 ↓
HLS
 ↓
Thumbnail
```

### Why asynchronous processing?

Video processing can take seconds or minutes.

The HTTP request should not remain open.

Instead:

``` text
POST /videos
     ↓
202 Accepted
     ↓
Job created
     ↓
Kafka
     ↓
Worker processes job
```

The client can query:

``` text
GET /videos/{id}/processing-status
```

Later this can become event-driven from the frontend.

------------------------------------------------------------------------

# 8. Phase 3 --- Transcription

## Goal

Convert video speech into timestamped text.

Pipeline:

``` text
Video
 ↓
FFmpeg
 ↓
WAV
 ↓
Speech-to-Text
 ↓
Timestamped transcript
 ↓
PostgreSQL
```

Example:

``` json
{
  "videoId": "VID-20260925-0001",
  "segments": [
    {
      "start": 0.0,
      "end": 5.2,
      "text": "Today we are going to learn Kafka."
    },
    {
      "start": 5.2,
      "end": 12.8,
      "text": "Kafka is an event streaming platform."
    }
  ]
}
```

Database concept:

``` text
transcript
----------------
id
video_id
language
version
status

transcript_segment
-------------------
id
transcript_id
sequence
start_time
end_time
text
```

### Important requirement

Transcript segments must preserve timestamps.

This enables:

``` text
Search result
     ↓
Transcript segment
     ↓
Video timestamp
     ↓
Player seeks to 32:14
```

------------------------------------------------------------------------

# 9. Phase 4 --- Concept Extraction

## Goal

Transform raw transcript text into structured knowledge.

Input:

``` text
Kafka is an event streaming platform.
Kafka producers publish records.
Consumers read records from topics.
Consumer groups allow parallel processing.
```

Output:

``` text
Kafka
 ├── Event Streaming
 ├── Producer
 ├── Consumer
 ├── Topic
 └── Consumer Group
```

Relationships:

``` text
Kafka
 ├── HAS_CONCEPT → Producer
 ├── HAS_CONCEPT → Consumer
 └── HAS_CONCEPT → Topic

Consumer Group
 └── ENABLES → Parallel Processing
```

### Important distinction

A concept is not necessarily the same as a keyword.

For example:

``` text
"database connection pooling"
```

may represent one concept even though it contains multiple words.

------------------------------------------------------------------------

# 10. Knowledge Graph Storage

Initially, do not introduce Neo4j.

Use PostgreSQL.

### Tables

``` text
concept
----------------
id
name
slug
description
type
created_at
updated_at

concept_relationship
---------------------
id
source_concept_id
target_concept_id
relationship_type
confidence

video_concept
-------------
video_id
concept_id
confidence

transcript_segment_concept
--------------------------
segment_id
concept_id
confidence
```

### Example

``` text
concept
--------------------------------
1 | Kafka
2 | Producer
3 | Consumer
4 | Topic
5 | Consumer Group
```

Relationship:

``` text
1 → 2 → HAS_CONCEPT
1 → 3 → HAS_CONCEPT
1 → 4 → HAS_CONCEPT
3 → 5 → USED_BY
```

------------------------------------------------------------------------

# 11. Graph Algorithms

This is where DSA becomes part of the real application.

Initially implement graph traversal yourself.

Useful algorithms:

### BFS

Use for:

``` text
Find concepts within N relationships.
```

Example:

``` text
Kafka
 ↓
Consumer
 ↓
Consumer Group
```

### DFS

Use for:

``` text
Explore concept hierarchy.
```

### Shortest path

Question:

``` text
How are Java and Kafka connected?
```

Possible result:

``` text
Java
 ↓
Concurrency
 ↓
Distributed Systems
 ↓
Messaging
 ↓
Kafka
```

### Topological sorting

Useful for learning prerequisites:

``` text
Concurrency
 ↓
Distributed Systems
 ↓
Messaging
 ↓
Kafka
 ↓
Kafka Streams
```

### Cycle detection

Important because prerequisite relationships should not contain invalid
cycles.

------------------------------------------------------------------------

# 12. Phase 5 --- Knowledge Search

## Goal

Allow users to search educational knowledge rather than just video
titles.

Example:

> Where does this course explain Kafka consumer groups?

Processing:

``` text
Query
 ↓
Keyword search
 ↓
Concept lookup
 ↓
Graph expansion
 ↓
Transcript search
 ↓
Ranking
 ↓
Timestamp
```

Example response:

``` text
Kafka Consumer Groups

Course: Kafka Fundamentals

32:14

"Consumer groups allow multiple consumers
to process partitions in parallel."
```

The user clicks the result and the video starts around:

``` text
32:14
```

------------------------------------------------------------------------

# 13. Search Evolution

Do not immediately introduce a complex search infrastructure.

Build search progressively.

## Level 1 --- PostgreSQL

Use:

-   B-tree indexes
-   PostgreSQL full-text search
-   `tsvector`
-   `tsquery`
-   trigram indexes where appropriate

## Level 2 --- Concept-aware search

``` text
Query
 ↓
Concept
 ↓
Related concepts
 ↓
Transcript
```

## Level 3 --- Semantic search

Introduce embeddings.

``` text
Text
 ↓
Embedding model
 ↓
Vector
 ↓
Vector database
```

Then combine:

``` text
Keyword score
+
Concept relevance
+
Vector similarity
+
Content quality
```

This creates a hybrid search system.

------------------------------------------------------------------------

# 14. Phase 6 --- Semantic Search and RAG

## Goal

Allow users to ask questions about the educational content.

Example:

> What happens when a Kafka consumer dies?

System:

``` text
Question
 ↓
Query embedding
 ↓
Vector search
 ↓
Concept expansion
 ↓
Keyword search
 ↓
Ranking
 ↓
Relevant transcript chunks
 ↓
Context construction
 ↓
LLM
 ↓
Answer + citations
```

Example answer structure:

``` text
When a consumer in a consumer group fails,
Kafka detects the failure and the group can
rebalance partitions among the remaining consumers.

Sources:

Kafka Fundamentals
32:14
Consumer Groups

Kafka Fundamentals
41:52
Partition Rebalancing
```

The important engineering principle is:

> The LLM should answer from retrieved project knowledge, not from an
> unrestricted prompt.

------------------------------------------------------------------------

# 15. RAG Components

Build the RAG system as separate stages.

``` text
Question
   ↓
Query preprocessing
   ↓
Retriever
   ↓
Candidate documents
   ↓
Reranker
   ↓
Context builder
   ↓
LLM
   ↓
Citation builder
```

### Retrieval sources

Potentially combine:

``` text
Transcript
Concepts
Concept relationships
Course metadata
Learning history
```

### Chunking

A transcript should not simply be split every N characters.

Consider:

-   sentence boundaries
-   timestamp boundaries
-   topic boundaries
-   chapter boundaries
-   overlapping context

Every chunk should retain:

``` text
video_id
segment_ids
start_time
end_time
concept_ids
text
embedding
```

------------------------------------------------------------------------

# 16. Phase 7 --- Learning Graph

The knowledge graph becomes more than search.

It can model prerequisites.

Example:

``` text
Kafka
 │
 ├── prerequisite → Distributed Systems
 │                      │
 │                      └── prerequisite → Concurrency
 │
 ├── prerequisite → Networking
 │
 └── related → Event Driven Architecture
```

Suppose the user already knows:

``` text
Java
Spring Boot
REST
SQL
```

and wants to learn:

``` text
Kafka
```

The system can construct a path through prerequisite concepts.

Example:

``` text
1. Java Concurrency
2. Distributed Systems
3. Messaging Fundamentals
4. Kafka Architecture
5. Kafka Producers
6. Kafka Consumers
7. Consumer Groups
8. Kafka Streams
9. Exactly-Once Processing
```

Initially, this path can be generated using graph algorithms.

AI can later improve explanations and personalization.

------------------------------------------------------------------------

# 17. User Learning Model

Eventually introduce user knowledge.

Possible entities:

``` text
user
concept
user_concept
learning_progress
learning_path
learning_path_item
```

Example:

``` text
user_concept
----------------
user_id
concept_id
status
confidence
last_reviewed_at
```

Statuses:

``` text
UNKNOWN
LEARNING
FAMILIAR
MASTERED
```

This allows:

``` text
User knowledge
      ↓
Missing prerequisites
      ↓
Knowledge graph
      ↓
Learning path
```

------------------------------------------------------------------------

# 18. Core Backend Services

Do not create all of these at the beginning.

Eventually:

## API Gateway

Responsibilities:

-   Authentication
-   Request routing
-   Correlation ID
-   Rate limiting
-   Security headers
-   Request filtering

## Auth/User Service

Responsibilities:

-   User
-   Authentication
-   Authorization
-   User learning profile

## Media Service

Responsibilities:

-   Video metadata
-   Upload sessions
-   Presigned URLs
-   Media assets
-   Video lifecycle

## Processing Workers

Responsibilities:

-   FFmpeg
-   HLS
-   thumbnails
-   audio extraction
-   transcription

## Knowledge Service

Responsibilities:

-   Concepts
-   Relationships
-   Knowledge graph
-   Graph traversal
-   Prerequisites

## Search Service

Responsibilities:

-   Keyword search
-   Concept search
-   Semantic search
-   Ranking

## Learning Service

Responsibilities:

-   User progress
-   Learning paths
-   Recommendations

## AI/RAG Service

Responsibilities:

-   Query processing
-   Retrieval
-   Context construction
-   LLM interaction
-   Citation generation

------------------------------------------------------------------------

# 19. Recommended Technology Responsibilities

Technology       Responsibility
  ---------------- -----------------------------------------
Java             Core backend services
Spring Boot      REST APIs and application framework
PostgreSQL       Transactional data and initial graph
Redis            Cache, rate limiting, short-lived state
Kafka            Event-driven processing
MinIO            Video/object storage
FFmpeg           Media processing
Python           AI/ML/transcription workers
Vector store     Semantic search
Vue              Frontend
Docker           Local development
Kubernetes       Deployment
Prometheus       Metrics
Grafana          Dashboards
Loki             Logs
Tempo            Distributed tracing
OpenTelemetry    Observability
GitHub Actions   CI/CD

------------------------------------------------------------------------

# 20. Important Distributed-System Patterns

LearnGraph should deliberately use production-oriented patterns.

## Outbox

Problem:

``` text
DB transaction succeeds
Kafka publish fails
```

Solution:

``` text
DB transaction
 ├── Business data
 └── Outbox event
          ↓
       Publisher
          ↓
        Kafka
```

## Idempotency

Every important event should have an identifier.

Example:

``` text
event_id
event_type
aggregate_id
occurred_at
correlation_id
```

Consumers must safely process duplicates.

------------------------------------------------------------------------

# 21. Retry

Transient failure:

``` text
Worker
 ↓
External STT service
 ↓
Timeout
```

Retry:

``` text
1st attempt
 ↓
backoff
 ↓
2nd attempt
 ↓
backoff
 ↓
3rd attempt
 ↓
DLQ
```

Avoid infinite retry loops.

------------------------------------------------------------------------

# 22. Dead Letter Queue

Messages that repeatedly fail should eventually be isolated.

``` text
Kafka Topic
    ↓
Consumer
    ↓
Failure
    ↓
Retry
    ↓
Retry
    ↓
DLQ
```

Store enough metadata to diagnose the problem.

------------------------------------------------------------------------

# 23. Concurrency

The project should become a practical playground for Java concurrency.

Potential workloads:

``` text
Process multiple videos
Process transcript chunks
Generate embeddings
Search multiple sources
Process Kafka partitions
Generate thumbnails
```

Study and compare:

``` text
Platform threads
ExecutorService
CompletableFuture
Structured concurrency concepts
Virtual threads
Kafka consumer concurrency
Batch processing
```

Do not assume more concurrency always means better performance.

Measure:

``` text
CPU
Memory
DB connections
Kafka lag
Throughput
Latency
External API limits
```

------------------------------------------------------------------------

# 24. Database Concurrency and Data Integrity

Important because multiple workers may update the same data.

Examples:

``` text
Worker A → video status
Worker B → video status
```

Potential techniques:

-   optimistic locking
-   atomic updates
-   database constraints
-   unique indexes
-   idempotency keys
-   transactions
-   row-level locking where appropriate

Avoid using a shared transaction across unrelated asynchronous tasks.

------------------------------------------------------------------------

# 25. Redis Usage

Redis should solve real problems.

Potential uses:

### Cache

``` text
Popular concept
Popular search
Video metadata
Learning path
```

### Rate limiting

``` text
User → API Gateway → Redis RateLimiter
```

### Distributed coordination

Only where genuinely required.

### Short-lived processing state

For example:

``` text
upload session
temporary token
job progress
```

Do not use Redis as the primary source of truth for important business
data.

------------------------------------------------------------------------

# 26. Kafka Event Model

Example topics:

``` text
media.video.uploaded
media.video.processing.started
media.video.processing.completed
media.video.processing.failed

transcription.requested
transcription.completed
transcription.failed

knowledge.concept.extracted
knowledge.graph.updated

search.embedding.requested
search.embedding.completed
```

Event example:

``` json
{
  "eventId": "evt-123",
  "eventType": "VideoUploaded",
  "aggregateId": "VID-20260925-0001",
  "occurredAt": "2026-09-25T10:00:00Z",
  "correlationId": "req-456",
  "payload": {
    "videoId": "VID-20260925-0001"
  }
}
```

------------------------------------------------------------------------

# 27. Observability

Every service should eventually expose:

## Metrics

Examples:

``` text
video_processing_duration
transcription_duration
kafka_consumer_lag
search_latency
rag_latency
embedding_requests
failed_processing_jobs
```

## Logs

Use structured logs.

Include:

``` text
timestamp
service
level
trace_id
span_id
correlation_id
event_id
video_id
message
```

## Tracing

Example:

``` text
HTTP Request
   ↓
Media Service
   ↓
Kafka
   ↓
Processing Worker
   ↓
Transcription
   ↓
Knowledge Service
```

A distributed trace should make the full workflow understandable.

------------------------------------------------------------------------

# 28. Failure Scenarios to Test

A major part of the project should be deliberately breaking things.

Test:

``` text
Kafka unavailable
PostgreSQL unavailable
Redis unavailable
MinIO unavailable
Worker crashes
Duplicate Kafka event
Out-of-order event
Slow database
Slow external AI service
Network timeout
Partial processing
Application restart
Consumer rebalance
```

For each failure ask:

``` text
What happens?
What data is lost?
What state remains?
Can processing resume?
Is the operation idempotent?
How does the user recover?
```

------------------------------------------------------------------------

# 29. Performance Testing

Measure rather than assume.

Important metrics:

``` text
Requests/sec
P50 latency
P95 latency
P99 latency
Kafka throughput
Kafka consumer lag
Database query time
Connection pool utilization
CPU
Memory
GC
Redis hit rate
Search latency
RAG latency
```

Test progressively:

``` text
10 videos
100 videos
1,000 videos
10,000 videos
```

The goal is not to pretend the system supports millions of users.

The goal is to understand where the bottlenecks appear.

------------------------------------------------------------------------

# 30. Suggested 3--4 Month Roadmap

## Month 1 --- Foundation

### Week 1

Project setup:

-   Git repository
-   Gradle/Maven structure
-   Docker Compose
-   PostgreSQL
-   MinIO
-   Kafka
-   Redis
-   basic Spring Boot service

### Week 2

Media domain:

-   Video aggregate
-   Video lifecycle
-   Media assets
-   database schema
-   Flyway
-   REST API

### Week 3

Upload:

-   Upload session
-   presigned URL
-   MinIO
-   upload completion
-   ObjectCreated event

### Week 4

Processing:

-   Kafka consumer
-   FFmpeg
-   HLS
-   thumbnail
-   processing status
-   retries

End of Month 1:

``` text
Upload Video
     ↓
MinIO
     ↓
Kafka
     ↓
Worker
     ↓
HLS + Thumbnail
     ↓
Playable Video
```

------------------------------------------------------------------------

# 31. Month 2 --- Knowledge Extraction

## Week 5

Audio/transcription:

-   FFmpeg audio extraction
-   STT integration
-   transcript model
-   timestamped segments

## Week 6

Transcript search:

-   PostgreSQL full-text search
-   indexes
-   timestamp navigation
-   search API

## Week 7

Concept extraction:

-   concept model
-   LLM/AI extraction
-   confidence
-   transcript-to-concept mapping

## Week 8

Knowledge graph:

-   concept relationships
-   graph traversal
-   BFS
-   DFS
-   shortest path
-   prerequisite relationships

End of Month 2:

``` text
Video
 ↓
Transcript
 ↓
Concepts
 ↓
Knowledge Graph
 ↓
Search
 ↓
Video Timestamp
```

------------------------------------------------------------------------

# 32. Month 3 --- AI and Learning

## Week 9

Semantic search:

-   chunking
-   embeddings
-   vector storage
-   similarity search

## Week 10

Hybrid search:

``` text
Keyword
+
Concept
+
Vector
```

Implement ranking.

## Week 11

RAG:

-   retrieval
-   context construction
-   LLM
-   citations
-   source timestamps

## Week 12

Learning graph:

-   user knowledge
-   prerequisites
-   learning path
-   progress
-   graph algorithms

End of Month 3:

``` text
User Question
 ↓
Hybrid Search
 ↓
Knowledge Graph
 ↓
Vector Search
 ↓
RAG
 ↓
Answer + Video Citations
```

------------------------------------------------------------------------

# 33. Month 4 --- Production Engineering

Month 4 is optional if the core product takes longer.

## Week 13

Reliability:

-   idempotency
-   outbox
-   retries
-   DLQ
-   circuit breaker
-   timeout
-   backpressure

## Week 14

Concurrency:

-   virtual threads
-   executor tuning
-   Kafka concurrency
-   batch processing
-   database connection limits

## Week 15

Observability:

-   OpenTelemetry
-   Prometheus
-   Grafana
-   Loki
-   Tempo
-   dashboards
-   alerts

## Week 16

Deployment:

``` text
Docker
 ↓
GitHub Actions
 ↓
GHCR
 ↓
Kubernetes
```

Add:

-   health checks
-   readiness/liveness
-   resource limits
-   rolling deployment
-   configuration/secrets
-   basic autoscaling

------------------------------------------------------------------------

# 34. What NOT to Build Initially

Avoid these until the core system works:

``` text
❌ Microservice for every table
❌ Neo4j
❌ Kubernetes on day one
❌ Complex AI agents
❌ Multi-region deployment
❌ Elasticsearch immediately
❌ Complex recommendation engine
❌ Real-time collaborative learning
❌ Billing/subscriptions
❌ Social network features
```

The project is primarily about:

``` text
Backend engineering
+
Distributed systems
+
Knowledge processing
+
AI-assisted search
```

------------------------------------------------------------------------

# 35. Suggested Initial Repository Structure

Start simple.

``` text
learn-graph/
│
├── services/
│   ├── media-service/
│   ├── learning-service/
│   └── gateway/
│
├── workers/
│   ├── media-worker/
│   ├── transcription-worker/
│   └── knowledge-worker/
│
├── frontend/
│   └── learn-graph-ui/
│
├── infra/
│   ├── docker/
│   ├── kafka/
│   ├── postgres/
│   ├── minio/
│   ├── redis/
│   └── observability/
│
├── docs/
│   ├── architecture/
│   ├── adr/
│   └── api/
│
└── README.md
```

The exact service boundaries can change as the system evolves.

------------------------------------------------------------------------

# 36. DDD Approach

Use DDD where it improves the model.

Do not turn every class into a DDD abstraction.

Important aggregates initially:

``` text
Video
UploadSession
ProcessingJob
Transcript
Concept
LearningPath
```

Example:

``` text
Video
 ├── VideoId
 ├── Title
 ├── Description
 ├── Status
 ├── Assets
 └── lifecycle methods
```

Business rules should live around the domain rather than being scattered
through controllers.

Keep the architecture practical.

A simpler DDD structure is preferred over introducing excessive
hexagonal/ports-and-adapters complexity for every service.

------------------------------------------------------------------------

# 37. Database Design Principles

Use PostgreSQL as the transactional source of truth.

Important techniques:

-   Foreign keys
-   Unique constraints
-   Check constraints
-   Indexes
-   Partial indexes
-   Full-text search
-   Transactions
-   Optimistic locking
-   Pagination
-   Batch operations

For large datasets, investigate:

``` text
EXPLAIN
EXPLAIN ANALYZE
query plans
index selectivity
connection pool sizing
vacuum
partitioning
```

------------------------------------------------------------------------

# 38. Security

Eventually implement:

-   JWT validation
-   issuer validation
-   audience validation
-   expiration validation
-   `nbf` validation
-   role/permission checks
-   request rate limiting
-   object access control
-   signed media URLs
-   input validation
-   upload size limits
-   file type validation

Do not trust:

``` text
X-User-Id
X-User-Role
```

headers supplied directly by clients.

If a gateway injects identity headers, strip untrusted incoming versions
first.

------------------------------------------------------------------------

# 39. API Examples

### Create video

``` http
POST /api/videos
```

Response:

``` json
{
  "videoId": "VID-20260925-0001",
  "status": "CREATED"
}
```

### Create upload URL

``` http
POST /api/videos/{videoId}/upload-url
```

### Search

``` http
GET /api/search?q=consumer+groups
```

### Concept

``` http
GET /api/concepts/{conceptId}
```

### Related concepts

``` http
GET /api/concepts/{conceptId}/related
```

### Learning path

``` http
GET /api/learning-paths?target=Kafka
```

### Ask question

``` http
POST /api/ask
```

Request:

``` json
{
  "question": "What happens when a Kafka consumer dies?"
}
```

------------------------------------------------------------------------

# 40. Example End-to-End Scenario

Suppose the user uploads:

``` text
Kafka Fundamentals.mp4
```

The system processes it.

### Step 1

Media service creates:

``` text
VID-20260925-0001
```

### Step 2

User uploads to:

``` text
MinIO
```

### Step 3

MinIO emits:

``` text
ObjectCreated
```

### Step 4

Kafka receives:

``` text
media.video.uploaded
```

### Step 5

Worker processes the video:

``` text
HLS
Thumbnail
Audio
```

### Step 6

Transcription:

``` text
00:00 Introduction
05:20 Kafka Producers
12:30 Kafka Topics
20:15 Partitions
32:14 Consumer Groups
41:52 Rebalancing
```

### Step 7

Concept extraction:

``` text
Kafka
Producer
Topic
Partition
Consumer
Consumer Group
Rebalancing
```

### Step 8

Graph:

``` text
Kafka
 ├── Topic
 │     └── Partition
 │
 └── Consumer
       └── Consumer Group
              └── Rebalancing
```

### Step 9

User asks:

> Where are consumer groups explained?

Search returns:

``` text
Kafka Fundamentals
32:14
```

### Step 10

User asks:

> What happens when one consumer crashes?

RAG retrieves:

``` text
Consumer Groups
Rebalancing
Partition Assignment
```

Then answers with citations:

``` text
32:14
41:52
```

------------------------------------------------------------------------

# 41. Portfolio Value

The final project should allow you to explain concrete engineering
decisions.

For example:

### Why Kafka?

Because video processing is asynchronous and independently scalable.

### Why MinIO?

Because large binary files should not be stored inside PostgreSQL.

### Why PostgreSQL?

Because metadata, transactional state, relationships, and initial graph
storage fit well together.

### Why Redis?

For low-latency caching and rate limiting.

### Why Python?

For AI/ML ecosystem integration while keeping the core backend in Java.

### Why Kafka instead of direct HTTP between workers?

To decouple producers and consumers and support asynchronous processing
and retry.

### Why Outbox?

To avoid inconsistent database/event publication.

### Why idempotency?

Because distributed systems can deliver duplicate messages.

### Why graph algorithms?

Because prerequisite and relationship traversal is a graph problem.

### Why vector search?

Because keyword matching cannot capture semantic similarity.

### Why RAG?

To ground generated answers in the platform's actual educational
content.

------------------------------------------------------------------------

# 42. Questions You Should Be Able to Answer After the Project

By the end, you should be able to explain:

## Java

-   When to use virtual threads
-   Executor sizing
-   CompletableFuture
-   concurrency hazards
-   synchronization
-   backpressure

## Spring Boot

-   transaction boundaries
-   async processing
-   Kafka integration
-   security
-   observability
-   configuration
-   testing

## PostgreSQL

-   indexes
-   transactions
-   isolation
-   locking
-   optimistic concurrency
-   query optimization
-   connection pooling

## Kafka

-   partitions
-   consumer groups
-   ordering
-   retries
-   rebalancing
-   offsets
-   delivery semantics
-   idempotent consumers

## Redis

-   cache-aside
-   TTL
-   distributed rate limiting
-   cache invalidation
-   Bloom filters

## Distributed Systems

-   eventual consistency
-   idempotency
-   retries
-   timeouts
-   circuit breakers
-   outbox
-   Saga
-   DLQ
-   backpressure

## AI

-   embeddings
-   chunking
-   vector search
-   hybrid retrieval
-   reranking
-   RAG
-   hallucination mitigation
-   evaluation

## Graphs

-   BFS
-   DFS
-   shortest path
-   topological sorting
-   cycle detection
-   prerequisite traversal

## Operations

-   Docker
-   Kubernetes
-   CI/CD
-   metrics
-   logs
-   tracing
-   alerting

------------------------------------------------------------------------

# 43. Definition of Done

The project should not be considered complete just because the APIs
work.

A meaningful final milestone is:

``` text
User
 ↓
Uploads video
 ↓
Video stored safely
 ↓
Asynchronous processing
 ↓
HLS generated
 ↓
Transcript generated
 ↓
Concepts extracted
 ↓
Knowledge graph updated
 ↓
Search works
 ↓
Semantic search works
 ↓
RAG answers questions
 ↓
Answers cite timestamps
 ↓
Learning path generated
 ↓
Failures are recoverable
 ↓
System is observable
 ↓
Tests exist
 ↓
Docker deployment works
 ↓
CI/CD works
```

------------------------------------------------------------------------

# 44. Testing Strategy

Use multiple testing levels.

## Unit tests

Test:

``` text
Domain rules
Graph algorithms
Ranking
Chunking
State transitions
```

## Integration tests

Test:

``` text
PostgreSQL
Kafka
Redis
MinIO
```

Use Testcontainers where appropriate.

## Contract/API tests

Verify service contracts.

## End-to-end tests

Test:

``` text
Upload
 → Process
 → Transcript
 → Concept
 → Search
```

## Failure tests

Intentionally kill:

``` text
Kafka
Database
Worker
Redis
MinIO
```

and verify recovery.

------------------------------------------------------------------------

# 45. Architecture Decision Records

Keep an ADR folder.

Examples:

``` text
ADR-001-use-postgresql-for-knowledge-graph.md
ADR-002-use-kafka-for-processing-pipeline.md
ADR-003-use-minio-for-video-storage.md
ADR-004-use-outbox-for-domain-events.md
ADR-005-use-python-for-ai-workers.md
ADR-006-start-with-postgresql-before-graph-database.md
ADR-007-use-hybrid-search.md
```

Each ADR should contain:

``` text
Context
Decision
Alternatives
Trade-offs
Consequences
```

This is valuable because the project demonstrates not only
implementation but engineering reasoning.

------------------------------------------------------------------------

# 46. Key Principle for the Entire Project

Do not optimize for the number of technologies.

Optimize for understanding.

For every technology ask:

``` text
What problem does this solve?
Why do I need it?
What happens without it?
What are the trade-offs?
How does it fail?
How do I measure it?
```

For every distributed component ask:

``` text
What happens if it crashes?
What happens if the message is duplicated?
What happens if the message arrives late?
What happens if the database succeeds but Kafka fails?
What happens if Kafka succeeds but the consumer crashes?
```

For every performance optimization ask:

``` text
What is the bottleneck?
How did I measure it?
What changed?
What new bottleneck appeared?
```

------------------------------------------------------------------------

# 47. Final Project Evolution

The complete journey should look like:

``` text
                   LEARNGRAPH
                       │
                       ▼
                 Video Platform
                       │
                       ▼
               Event-driven Media
                       │
                       ▼
                Async Processing
                       │
                       ▼
                  Transcription
                       │
                       ▼
               Concept Extraction
                       │
                       ▼
                Knowledge Graph
                       │
                       ▼
                  Text Search
                       │
                       ▼
                 Semantic Search
                       │
                       ▼
                       RAG
                       │
                       ▼
               Learning Graph
                       │
                       ▼
              Concurrent Processing
                       │
                       ▼
                  Reliability
                       │
                       ▼
                 Observability
                       │
                       ▼
                CI/CD + Kubernetes
```

The project therefore becomes both:

1.  **A usable educational product**
2.  **A long-term backend engineering laboratory**

------------------------------------------------------------------------

# 48. First Milestone

When returning to this project after a break, do **not** start with AI.

Start here:

``` text
Media Service
      ↓
Create Video
      ↓
Generate Presigned URL
      ↓
Upload to MinIO
      ↓
ObjectCreated Event
      ↓
Kafka
      ↓
Processing Worker
      ↓
FFmpeg
      ↓
HLS + Thumbnail
      ↓
Video READY
```

Once this works reliably, continue to:

``` text
Transcript
 ↓
Concepts
 ↓
Knowledge Graph
 ↓
Search
 ↓
Semantic Search
 ↓
RAG
 ↓
Learning Paths
```

This sequence is the project's main development path.

------------------------------------------------------------------------

# 49. One-Sentence Project Definition

> **LearnGraph is an event-driven video learning platform that
> transforms educational videos into timestamped transcripts, searchable
> concepts, a connected knowledge graph, semantic search results,
> grounded RAG answers, and personalized learning paths.**