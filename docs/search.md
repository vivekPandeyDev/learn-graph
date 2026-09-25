# LearnGraph --- Search Architecture

## 1. Goal

Search should answer more than:

> Which video contains this exact word?

It should eventually answer:

> Which part of which video explains this concept or a closely related
> concept?

------------------------------------------------------------------------

# 2. Search Evolution

``` text
PostgreSQL FTS
      ↓
Concept-aware search
      ↓
Semantic search
      ↓
Hybrid search
      ↓
RAG retrieval
```

------------------------------------------------------------------------

# 3. Phase 1 --- PostgreSQL Full-Text Search

Store searchable transcript text.

Use:

``` text
tsvector
tsquery
GIN index
```

Pipeline:

``` text
Query
 ↓
PostgreSQL FTS
 ↓
Transcript segments
```

This is the first implementation.

------------------------------------------------------------------------

# 4. Search Result Model

A search result should identify the exact learning location.

``` json
{
  "videoId": "VID-123",
  "segmentId": "seg-42",
  "startMs": 120000,
  "endMs": 128000,
  "text": "A consumer group...",
  "score": 0.88
}
```

This allows the UI to jump directly to the relevant timestamp.

------------------------------------------------------------------------

# 5. Phase 2 --- Concept Search

Query:

``` text
"consumer failure"
```

Concept expansion:

``` text
Consumer
Consumer Group
Rebalancing
Partition Assignment
Offset
```

Then search related transcript segments.

------------------------------------------------------------------------

# 6. Phase 3 --- Semantic Search

Transcript segments are chunked and embedded.

``` text
Transcript
 ↓
Chunking
 ↓
Embedding model
 ↓
Vector
 ↓
Vector storage
```

Query:

``` text
"Why do Kafka consumers get reassigned?"
```

can retrieve:

``` text
"Consumer group rebalancing occurs when..."
```

even if the exact words differ.

------------------------------------------------------------------------

# 7. Chunking

A chunk should preserve meaningful context.

Avoid arbitrary huge chunks.

Store:

``` text
chunkId
videoId
segmentIds
startMs
endMs
text
embedding
conceptIds
```

A chunk should be traceable back to the original transcript.

------------------------------------------------------------------------

# 8. Hybrid Search

Final architecture:

``` text
                    Query
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
     Keyword       Concept        Vector
      Search        Search        Search
        │             │             │
        └─────────────┼─────────────┘
                      ▼
                  Candidate Set
                      ↓
                   Ranking
                      ↓
                  Top Results
```

------------------------------------------------------------------------

# 9. Ranking

A conceptual ranking score can combine:

``` text
keywordScore
conceptScore
semanticScore
conceptImportance
videoRelevance
freshness
```

Example:

``` text
finalScore =
    0.30 * keywordScore
  + 0.25 * conceptScore
  + 0.35 * semanticScore
  + 0.10 * importance
```

The actual weights should be determined experimentally rather than
treated as permanent constants.

------------------------------------------------------------------------

# 10. Filtering

Potential filters:

``` text
Course
Video
Concept
Language
Difficulty
Duration
Creator
```

Filtering should happen as early as practical to reduce candidate
volume.

------------------------------------------------------------------------

# 11. Search Caching

Redis can cache popular queries:

``` text
search:{normalized-query}:{filters}
```

Use a TTL.

Do not cache indefinitely because the underlying knowledge graph and
transcripts can change.

------------------------------------------------------------------------

# 12. Search Consistency

Search indexes are derived data.

Therefore:

``` text
PostgreSQL = source of truth
Search index = derived
Vector store = derived
```

If indexing fails:

``` text
Knowledge remains persisted
Indexing can be retried
```

------------------------------------------------------------------------

# 13. Indexing Pipeline

``` text
Transcript Completed
       ↓
Chunking
       ↓
Keyword Index
       ↓
Concept Mapping
       ↓
Embedding
       ↓
Vector Index
```

Events make each step independently retryable.

------------------------------------------------------------------------

# 14. Search Failure

Example:

``` text
Concept persisted
      ↓
Embedding service fails
```

Expected state:

``` text
Concept = AVAILABLE
Embedding = PENDING
```

The system should retry indexing without rebuilding the concept.

------------------------------------------------------------------------

# 15. Search Evaluation

Create a test dataset:

``` text
Query
Expected videos
Expected concepts
Expected segments
```

Metrics:

``` text
Precision@K
Recall@K
MRR
NDCG
Latency
```

This turns search quality into something measurable.

------------------------------------------------------------------------

# 16. Search API

``` http
GET /api/v1/search?q=kafka+consumer&size=20
```

Future:

``` text
mode=keyword
mode=semantic
mode=hybrid

conceptId=...
videoId=...
```

------------------------------------------------------------------------

# 17. Search Architecture Evolution

Start:

``` text
PostgreSQL
```

Then:

``` text
PostgreSQL + embeddings
```

Then, only if needed:

``` text
Dedicated search/vector infrastructure
```

Do not introduce Elasticsearch/OpenSearch simply because it is common in
production systems.
