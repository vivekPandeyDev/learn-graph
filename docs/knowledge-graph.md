# LearnGraph --- Knowledge Graph

## 1. Purpose

The knowledge graph is the core differentiator of LearnGraph.

Instead of treating a video as one searchable document, the system
extracts:

``` text
Concepts
Relationships
Prerequisites
Video references
Transcript references
```

Example:

``` text
Kafka
 ├── Consumer
 │    └── Consumer Group
 ├── Producer
 └── Partition
       └── Offset
```

------------------------------------------------------------------------

# 2. Why PostgreSQL First?

The first implementation uses PostgreSQL because:

-   The project already needs PostgreSQL.
-   Concepts and relationships have transactional requirements.
-   SQL supports joins and recursive queries.
-   Application-level graph algorithms are sufficient for the initial
    graph size.
-   Operational complexity stays low.

A graph database can be evaluated later using measured requirements.

------------------------------------------------------------------------

# 3. Core Model

Conceptually:

``` text
Concept
  │
  ├── ConceptRelationship
  │
  ├── VideoConcept
  │
  └── TranscriptSegmentConcept
```

------------------------------------------------------------------------

# 4. Concept

Example:

``` text
Concept
---------
id
name
normalized_name
description
type
created_at
updated_at
```

Potential types:

``` text
TECHNOLOGY
ALGORITHM
DESIGN_PATTERN
FRAMEWORK
PROTOCOL
DATABASE
LANGUAGE
CONCEPT
```

------------------------------------------------------------------------

# 5. Concept Relationship

``` text
ConceptRelationship
-------------------
id
source_concept_id
target_concept_id
relationship_type
confidence
source
created_at
```

Examples:

``` text
REQUIRES
RELATED_TO
PART_OF
IMPLEMENTS
ALTERNATIVE_TO
USED_WITH
```

Example:

``` text
Kafka Consumer
    --REQUIRES-->
Consumer Group
```

------------------------------------------------------------------------

# 6. Video Concept

A concept can occur in many videos.

``` text
Video
  │
  ├── VideoConcept
  │
  ▼
Concept
```

The mapping may contain:

``` text
video_id
concept_id
confidence
first_seen_ms
last_seen_ms
importance
```

------------------------------------------------------------------------

# 7. Transcript Segment

Transcript data should preserve timestamps.

``` text
TranscriptSegment
-----------------
id
video_id
start_ms
end_ms
text
sequence_number
```

Example:

``` text
120000 → 126000
"A consumer group is a set of consumers..."
```

------------------------------------------------------------------------

# 8. Segment-to-Concept Mapping

``` text
TranscriptSegment
       │
       ▼
SegmentConcept
       │
       ▼
Concept
```

This enables:

``` text
Search concept
    ↓
Video
    ↓
Timestamp
```

------------------------------------------------------------------------

# 9. Concept Extraction Pipeline

``` text
Transcript
    ↓
Chunk
    ↓
LLM / NLP
    ↓
Candidate Concepts
    ↓
Normalization
    ↓
Deduplication
    ↓
Persist
```

Example:

``` text
"consumer groups"
"Kafka consumer group"
"consumer-group"
```

may normalize to:

``` text
Kafka Consumer Group
```

------------------------------------------------------------------------

# 10. Relationship Extraction

Relationship extraction should produce candidates:

``` json
{
  "source": "Consumer Group",
  "target": "Partition",
  "relationship": "ASSIGNS",
  "confidence": 0.89
}
```

The application should validate the relationship before persistence.

AI output should not automatically bypass domain constraints.

------------------------------------------------------------------------

# 11. Confidence

AI-generated knowledge should retain confidence.

Example:

``` text
Concept confidence = 0.94
Relationship confidence = 0.81
```

Confidence is metadata, not absolute truth.

It can later support:

``` text
Human review
Ranking
Conflict resolution
Quality evaluation
```

------------------------------------------------------------------------

# 12. Duplicate Concepts

Use normalized identifiers.

Example:

``` text
"Java Streams"
"java stream"
"Java Stream API"
```

can be candidates for the same canonical concept.

A normalization pipeline can use:

``` text
Lowercase
Whitespace normalization
Punctuation normalization
Alias dictionary
Semantic similarity
```

Human/admin review can resolve ambiguous cases.

------------------------------------------------------------------------

# 13. Graph Traversal

## BFS

Useful for:

``` text
Related concepts within N hops
```

Example:

``` text
Kafka
 ↓
Consumer
 ↓
Consumer Group
 ↓
Rebalancing
```

------------------------------------------------------------------------

## DFS

Useful for:

``` text
Exploring a concept hierarchy
```

------------------------------------------------------------------------

## Shortest Path

Question:

``` text
How is Kafka related to exactly-once processing?
```

Possible path:

``` text
Kafka
 ↓
Producer
 ↓
Transaction
 ↓
Exactly Once
```

------------------------------------------------------------------------

## Topological Sort

Useful for learning paths.

Example:

``` text
Java Basics
    ↓
Concurrency
    ↓
Thread Safety
    ↓
Distributed Systems
    ↓
Kafka
```

------------------------------------------------------------------------

# 14. Cycle Detection

Prerequisite relationships should generally be acyclic.

Invalid:

``` text
A REQUIRES B
B REQUIRES C
C REQUIRES A
```

Detect cycles before accepting prerequisite edges.

------------------------------------------------------------------------

# 15. Graph Query Strategy

Initial approach:

``` text
PostgreSQL query
      ↓
Application graph algorithm
      ↓
Result
```

Use recursive SQL where it is simpler and efficient.

Do not automatically load the entire graph into memory.

------------------------------------------------------------------------

# 16. Indexes

Important indexes:

``` text
concept(normalized_name)

concept_relationship(source_concept_id)
concept_relationship(target_concept_id)

video_concept(video_id)
video_concept(concept_id)

transcript_segment(video_id, sequence_number)
segment_concept(concept_id)
```

Unique constraints should prevent duplicate logical relationships.

------------------------------------------------------------------------

# 17. Knowledge Quality

The graph should support multiple sources:

``` text
AI extraction
Manual correction
Imported metadata
Future user feedback
```

Track provenance:

``` text
source_type
source_reference
confidence
created_at
updated_at
```

This becomes important when two sources disagree.

------------------------------------------------------------------------

# 18. Conflict Handling

Example:

``` text
Video A:
X requires Y

Video B:
X does not require Y
```

Do not silently overwrite one with the other.

Represent provenance and confidence.

Later the system can support:

``` text
SUPPORTED_BY
CONTRADICTED_BY
REVIEW_REQUIRED
```

------------------------------------------------------------------------

# 19. When to Consider Neo4j

Evaluate migration only when evidence shows:

``` text
Traversal latency is too high
Recursive queries become difficult
Graph size becomes very large
Graph query patterns dominate workload
```

The domain model should remain conceptually independent of the storage
engine.

------------------------------------------------------------------------

# 20. Knowledge Graph API Examples

``` text
GET /concepts/{id}

GET /concepts/{id}/related

GET /concepts/{id}/prerequisites

GET /concepts/path?from=A&to=B

GET /videos/{id}/concepts
```

------------------------------------------------------------------------

# 21. Knowledge Graph Evolution

``` text
Phase 1
Concept + relationship tables

Phase 2
Graph algorithms

Phase 3
Concept-aware search

Phase 4
Learning paths

Phase 5
Quality/conflict management

Phase 6
Evaluate graph database if required
```
