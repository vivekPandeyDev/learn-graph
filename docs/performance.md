# LearnGraph --- Performance Engineering

## 1. Philosophy

Performance work should follow:

``` text
Measure
 ↓
Find bottleneck
 ↓
Change
 ↓
Benchmark
 ↓
Compare
```

Avoid optimizing based only on intuition.

------------------------------------------------------------------------

# 2. Important Metrics

API:

``` text
RPS
P50
P95
P99
Error rate
```

Kafka:

``` text
Consumer lag
Throughput
Processing time
```

Database:

``` text
Query latency
Connection pool usage
CPU
Locks
Slow queries
```

Workers:

``` text
Jobs/minute
Average processing time
Failure rate
CPU
Memory
```

Search:

``` text
Retrieval latency
Ranking latency
P95/P99
Precision@K
Recall@K
```

RAG:

``` text
Retrieval latency
LLM latency
Token count
Cost
Citation accuracy
```

------------------------------------------------------------------------

# 3. Database Performance

Important practices:

``` text
Correct indexes
Batch writes
Avoid N+1
Pagination
Query plans
Connection pool tuning
```

Use:

``` sql
EXPLAIN ANALYZE
```

before making assumptions about slow queries.

------------------------------------------------------------------------

# 4. Connection Pools

Concurrency is constrained by the database.

Example:

``` text
500 virtual threads
        ↓
PostgreSQL pool = 30
        ↓
30 active database connections
```

Increasing application concurrency without considering the database can
make performance worse.

------------------------------------------------------------------------

# 5. Virtual Threads

Virtual threads are useful for many blocking I/O operations.

Good candidates:

``` text
HTTP calls
File operations
I/O-heavy workflows
```

But they do not make:

``` text
CPU-heavy FFmpeg
LLM inference
Database
```

infinitely scalable.

The underlying resource still limits throughput.

------------------------------------------------------------------------

# 6. Kafka Performance

Tune based on:

``` text
partition count
consumer count
batch size
poll configuration
message size
processing time
```

Do not create excessive partitions without an operational reason.

------------------------------------------------------------------------

# 7. Worker Scaling

Example:

``` text
Kafka lag increasing
        ↓
Increase worker replicas
```

But first determine whether the bottleneck is:

``` text
CPU
Memory
Database
External API
Disk
Network
```

------------------------------------------------------------------------

# 8. FFmpeg Workloads

Video processing is CPU/disk intensive.

Separate media processing workers from normal API services.

Monitor:

``` text
CPU
disk throughput
processing duration
source file size
output bitrate
```

------------------------------------------------------------------------

# 9. Search Performance

Reduce candidate volume:

``` text
Filters
 ↓
Keyword candidate set
 ↓
Concept candidate set
 ↓
Vector candidate set
 ↓
Ranking
```

Do not run expensive ranking over millions of candidates if early
filtering can reduce the set.

------------------------------------------------------------------------

# 10. RAG Performance

Typical latency:

``` text
Query processing
 + retrieval
 + ranking
 + context construction
 + LLM
```

The LLM may dominate latency.

Optimize by:

``` text
Smaller context
Better retrieval
Caching
Batch embeddings
Appropriate model selection
```

------------------------------------------------------------------------

# 11. Load Testing

Useful scenarios:

### API

``` text
100 concurrent users
500 concurrent users
```

### Upload

``` text
10 videos/min
100 videos/min
```

### Search

``` text
100 RPS
500 RPS
```

### Processing

Measure:

``` text
video processing time
transcription throughput
knowledge extraction throughput
```

Use realistic payload sizes.

------------------------------------------------------------------------

# 12. Bottleneck Documentation

For every major benchmark record:

``` text
Environment
Workload
Configuration
Result
Bottleneck
Change
New result
```

This makes performance improvements reproducible.
