# LearnGraph --- RAG Architecture

## 1. Purpose

Retrieval-Augmented Generation allows users to ask questions about the
knowledge contained in LearnGraph videos.

The central rule is:

> The LLM generates answers from retrieved project knowledge rather than
> acting as the knowledge source.

------------------------------------------------------------------------

# 2. High-Level Flow

``` text
User Question
      ↓
Query Processing
      ↓
Retrieval
 ┌────┼────┐
 ▼    ▼    ▼
FTS Concept Vector
 └────┼────┘
      ↓
Candidate Ranking
      ↓
Context Builder
      ↓
LLM
      ↓
Citation Mapper
      ↓
Answer
```

------------------------------------------------------------------------

# 3. Query Processing

Normalize:

``` text
Whitespace
Language
Terminology
```

Optionally extract:

``` text
Concepts
Entities
Intent
Filters
```

Example:

``` text
"Why does Kafka rebalance consumers?"
```

becomes:

``` text
Intent = explanation
Concepts = Kafka, Consumer, Consumer Group, Rebalancing
```

------------------------------------------------------------------------

# 4. Retrieval

Use multiple retrieval strategies.

### Keyword

Good for exact terminology.

### Concept

Good for knowledge relationships.

### Vector

Good for semantic similarity.

A hybrid strategy generally provides richer candidates than relying on
only one method.

------------------------------------------------------------------------

# 5. Candidate Ranking

Each candidate contains:

``` text
videoId
segmentId
text
startMs
endMs
conceptIds
scores
```

Rank candidates before sending them to the LLM.

This reduces irrelevant context and token usage.

------------------------------------------------------------------------

# 6. Context Construction

Context should preserve source information.

Example:

``` text
[Video: Kafka Fundamentals]
[Timestamp: 12:30 - 13:05]
[Concepts: Consumer Group, Rebalancing]

Consumer groups coordinate...
```

The LLM receives source metadata alongside text.

------------------------------------------------------------------------

# 7. Context Limits

Do not send every retrieved result.

Use:

``` text
Top K
deduplication
diversity
token budget
```

Example:

``` text
Retrieve 30
 ↓
Rank
 ↓
Select 8
 ↓
Build context
 ↓
LLM
```

------------------------------------------------------------------------

# 8. Citation Generation

The answer should preserve source references.

Example:

``` json
{
  "answer": "...",
  "citations": [
    {
      "videoId": "VID-123",
      "startMs": 720000,
      "endMs": 760000,
      "segmentId": "seg-88"
    }
  ]
}
```

The UI can convert this into:

``` text
Watch at 12:00
```

------------------------------------------------------------------------

# 9. Hallucination Control

The RAG service should:

-   Require retrieved evidence.
-   Keep context source identifiers.
-   Avoid unsupported claims.
-   Distinguish retrieved facts from generated explanations.
-   Return a limited answer when evidence is insufficient.

Potential response:

``` text
I couldn't find enough material in the available videos to answer this confidently.
```

------------------------------------------------------------------------

# 10. RAG Evaluation

Build a dataset:

``` text
Question
Expected source
Expected concepts
Expected answer characteristics
```

Evaluate:

``` text
Retrieval recall
Context relevance
Citation correctness
Answer faithfulness
Latency
Token usage
Cost
```

------------------------------------------------------------------------

# 11. RAG Observability

Record:

``` text
query
retrieval latency
candidate count
selected chunks
LLM latency
token count
model
failure reason
```

Avoid logging sensitive user content unnecessarily.

------------------------------------------------------------------------

# 12. RAG Caching

Potential cache layers:

``` text
Query embedding cache
Retrieval result cache
Repeated answer cache
```

Be careful with answer caching if:

``` text
knowledge changes
user permissions differ
scope differs
```

------------------------------------------------------------------------

# 13. RAG Failure Modes

### Retrieval failure

Return a useful "insufficient evidence" response.

### Vector service failure

Fallback to keyword/concept retrieval if appropriate.

### LLM timeout

Return an error or retry according to policy.

### Invalid LLM output

Validate structured output and retry or fail safely.

------------------------------------------------------------------------

# 14. RAG Architecture Boundary

The RAG service orchestrates:

``` text
Retrieval
Context
Generation
Citations
```

It should not own:

``` text
Video lifecycle
Transcript truth
Concept identity
Learning progress
Authentication
```

Those belong to their respective domains.

------------------------------------------------------------------------

# 15. Future RAG Features

Potential later features:

``` text
Compare two videos
Explain concept at beginner level
Generate quiz
Generate flashcards
Summarize learning gaps
Ask follow-up questions
Personalized explanations
```

These should be added only after retrieval quality is reliable.
