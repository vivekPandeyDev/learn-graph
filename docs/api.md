# LearnGraph --- API Design

## Purpose

This document defines the HTTP API conventions and initial endpoint
design for LearnGraph.

The API should expose business capabilities rather than database CRUD
operations.

------------------------------------------------------------------------

## 1. API Principles

-   REST for synchronous business operations.
-   Kafka for long-running asynchronous processing.
-   Pagination for potentially large collections.
-   Stable resource identifiers.
-   Explicit lifecycle/state transitions.
-   Consistent error responses.
-   Idempotency for retryable commands.
-   Correlation IDs for distributed tracing.

------------------------------------------------------------------------

## 2. Base URL

Development:

``` text
/api/v1
```

Example:

``` text
GET /api/v1/videos/{videoId}
```

------------------------------------------------------------------------

## 3. Authentication

Requests use:

``` http
Authorization: Bearer <JWT>
```

The gateway validates:

``` text
signature
iss
aud
exp
nbf
```

Services perform authorization based on the authenticated principal.

------------------------------------------------------------------------

## 4. Correlation ID

Clients may provide:

``` http
X-Correlation-Id: req-123
```

If absent, the gateway generates one.

The ID should propagate through:

``` text
HTTP → Kafka → Worker → HTTP/AI calls
```

------------------------------------------------------------------------

# 5. Video APIs

## Create video

``` http
POST /api/v1/videos
```

Request:

``` json
{
  "title": "Kafka Consumer Groups",
  "description": "Understanding consumer groups",
  "sourceFileName": "kafka-consumers.mp4"
}
```

Response:

``` json
{
  "videoId": "VID-20260925-000001",
  "status": "CREATED",
  "createdAt": "2026-09-25T10:00:00Z"
}
```

------------------------------------------------------------------------

## Get video

``` http
GET /api/v1/videos/{videoId}
```

Response should contain metadata, not the video binary.

------------------------------------------------------------------------

## Generate upload URL

``` http
POST /api/v1/videos/{videoId}/upload-url
```

Response:

``` json
{
  "uploadUrl": "...",
  "objectKey": "raw/VID-20260925-000001/source.mp4",
  "expiresAt": "2026-09-25T10:15:00Z"
}
```

The client uploads directly to object storage.

------------------------------------------------------------------------

## Complete upload

``` http
POST /api/v1/videos/{videoId}/upload-complete
```

The operation confirms that the client believes the upload is complete.
Object-storage events remain the authoritative processing trigger where
configured.

------------------------------------------------------------------------

## Get processing status

``` http
GET /api/v1/videos/{videoId}/processing
```

Example:

``` json
{
  "videoId": "VID-20260925-000001",
  "overallStatus": "TRANSCRIBING",
  "stages": {
    "MEDIA": "COMPLETED",
    "TRANSCRIPTION": "RUNNING",
    "KNOWLEDGE": "PENDING",
    "INDEXING": "PENDING"
  }
}
```

------------------------------------------------------------------------

# 6. Playback APIs

## Get playback information

``` http
GET /api/v1/videos/{videoId}/playback
```

Response:

``` json
{
  "videoId": "VID-20260925-000001",
  "masterPlaylistUrl": "...",
  "expiresAt": "2026-09-25T11:00:00Z"
}
```

The API returns a secure playback reference rather than proxying the
entire video through the application.

------------------------------------------------------------------------

# 7. Transcript APIs

## Get transcript

``` http
GET /api/v1/videos/{videoId}/transcript
```

Optional:

``` text
?from=120&to=300
```

Response:

``` json
{
  "videoId": "VID-20260925-000001",
  "segments": [
    {
      "segmentId": "seg-101",
      "startMs": 120000,
      "endMs": 126000,
      "text": "A Kafka consumer group..."
    }
  ]
}
```

------------------------------------------------------------------------

# 8. Knowledge APIs

## Get concepts for a video

``` http
GET /api/v1/videos/{videoId}/concepts
```

Example:

``` json
{
  "videoId": "VID-20260925-000001",
  "concepts": [
    {
      "conceptId": "concept-kafka-consumer",
      "name": "Kafka Consumer",
      "confidence": 0.96
    }
  ]
}
```

------------------------------------------------------------------------

## Get concept

``` http
GET /api/v1/concepts/{conceptId}
```

Returns:

``` text
Concept metadata
Relationships
Related videos
Prerequisites
```

------------------------------------------------------------------------

## Find relationship path

``` http
GET /api/v1/concepts/path
```

Parameters:

``` text
from=Kafka
to=Exactly-Once-Processing
```

Response:

``` json
{
  "path": [
    "Kafka",
    "Consumer",
    "Offset",
    "Transaction",
    "Exactly-Once-Processing"
  ]
}
```

------------------------------------------------------------------------

# 9. Search APIs

``` http
GET /api/v1/search
```

Parameters:

``` text
q=consumer group
page=0
size=20
```

Future filters:

``` text
conceptId
videoId
courseId
difficulty
language
```

Response:

``` json
{
  "query": "consumer group",
  "results": [
    {
      "videoId": "VID-20260925-000001",
      "segmentId": "seg-101",
      "startMs": 120000,
      "endMs": 126000,
      "score": 0.91,
      "matchedConcepts": [
        "Kafka Consumer Group"
      ]
    }
  ],
  "page": 0,
  "size": 20,
  "total": 42
}
```

------------------------------------------------------------------------

# 10. RAG API

``` http
POST /api/v1/ask
```

Request:

``` json
{
  "question": "Why does a Kafka consumer group rebalance?",
  "scope": {
    "courseId": "course-123"
  }
}
```

Response:

``` json
{
  "answer": "A rebalance occurs when group membership or partition ownership changes...",
  "citations": [
    {
      "videoId": "VID-20260925-000001",
      "segmentId": "seg-101",
      "startMs": 120000,
      "endMs": 126000
    }
  ]
}
```

The answer should be grounded in retrieved project content.

------------------------------------------------------------------------

# 11. Learning APIs

## Get learning profile

``` http
GET /api/v1/me/learning-profile
```

------------------------------------------------------------------------

## Mark concept as learned

``` http
POST /api/v1/me/concepts/{conceptId}/progress
```

Request:

``` json
{
  "status": "LEARNING"
}
```

------------------------------------------------------------------------

## Generate learning path

``` http
POST /api/v1/learning-paths
```

Request:

``` json
{
  "targetConcept": "Kafka Streams"
}
```

The service uses prerequisite relationships and user progress to
construct a path.

------------------------------------------------------------------------

# 12. Pagination

Use:

``` text
page
size
sort
```

Example:

``` text
?page=0&size=20&sort=createdAt,desc
```

Avoid unbounded collection endpoints.

------------------------------------------------------------------------

# 13. Error Format

Use a consistent structure:

``` json
{
  "type": "https://learn-graph.dev/errors/video-not-found",
  "title": "Video not found",
  "status": 404,
  "detail": "Video VID-20260925-000001 does not exist",
  "instance": "/api/v1/videos/VID-20260925-000001",
  "correlationId": "req-123"
}
```

------------------------------------------------------------------------

# 14. Idempotency

For commands that may be retried:

``` http
Idempotency-Key: 7f2c...
```

Useful for:

``` text
Create operations
Processing commands
Learning progress updates
```

The server should define which endpoints actually support idempotency
rather than accepting the header everywhere without semantics.

------------------------------------------------------------------------

# 15. API Lifecycle

Do not expose internal database models directly.

Prefer:

``` text
Controller
   ↓
Application Service
   ↓
Domain
   ↓
Repository
```

DTOs are API contracts.

------------------------------------------------------------------------

# 16. Initial API Priority

Build first:

``` text
POST   /videos
POST   /videos/{id}/upload-url
GET    /videos/{id}
GET    /videos/{id}/processing
GET    /videos/{id}/playback
GET    /videos/{id}/transcript
GET    /search
GET    /concepts/{id}
POST   /ask
```

Learning APIs can follow after the knowledge graph is stable.
