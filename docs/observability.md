# LearnGraph --- Observability

## 1. Goal

Observability should answer:

``` text
What happened?
Why did it happen?
Where did it happen?
How long did it take?
Which component failed?
```

Use:

``` text
Logs
Metrics
Traces
```

------------------------------------------------------------------------

# 2. Stack

``` text
Application
   │
OpenTelemetry / Micrometer
   │
 ┌─┼──────────┐
 ▼ ▼          ▼
Tempo Prometheus Loki
 └─┬──────────┘
   ▼
Grafana
```

------------------------------------------------------------------------

# 3. Tracing

Trace:

``` text
HTTP
 ↓
Service
 ↓
Database
 ↓
Kafka
 ↓
Worker
 ↓
External AI service
```

Important attributes:

``` text
service.name
environment
video.id
event.id
correlation.id
```

Avoid putting sensitive content into span attributes.

------------------------------------------------------------------------

# 4. Metrics

Application metrics:

``` text
http.server.requests
http.server.duration
jvm.memory.used
jvm.threads
```

Business metrics:

``` text
videos_uploaded_total
videos_processed_total
transcriptions_completed_total
concepts_extracted_total
search_requests_total
rag_requests_total
```

Kafka:

``` text
consumer_lag
processing_duration
```

------------------------------------------------------------------------

# 5. Logs

Structured logs should contain:

``` text
timestamp
level
service
traceId
spanId
correlationId
eventId
videoId
message
```

Example:

``` json
{
  "level": "INFO",
  "service": "media-worker",
  "videoId": "VID-123",
  "eventId": "evt-456",
  "message": "HLS processing completed"
}
```

------------------------------------------------------------------------

# 6. Correlation

A single user workflow should be searchable using:

``` text
correlationId
```

Example:

``` text
HTTP request
 ↓
Kafka event
 ↓
Worker
 ↓
STT
 ↓
Knowledge worker
```

All should be connected to the same distributed workflow.

------------------------------------------------------------------------

# 7. Dashboards

Initial Grafana dashboards:

### API

``` text
Request rate
Latency
Errors
```

### Kafka

``` text
Consumer lag
Throughput
Failures
```

### Workers

``` text
Jobs
Processing time
Failures
```

### Database

``` text
Connections
Latency
CPU
```

### RAG

``` text
Queries
Latency
Tokens
Errors
```

------------------------------------------------------------------------

# 8. Alerts

Examples:

``` text
Kafka lag above threshold
API error rate increasing
Database connection pool exhausted
Worker failure rate increasing
Processing queue continuously growing
```

Alerts should represent actionable problems.

------------------------------------------------------------------------

# 9. Tracing Failure Investigation

Example:

``` text
User reports:
"Video processing is stuck."

Trace:
request
 ↓
upload
 ↓
Kafka
 ↓
media-worker
 ↓
transcription.requested
 ↓
transcription-worker
 ↓
STT timeout
```

Now the failure can be localized instead of guessed.

------------------------------------------------------------------------

# 10. Observability Principles

-   Instrument important workflows.
-   Keep logs structured.
-   Use correlation IDs.
-   Measure business operations.
-   Avoid high-cardinality metric labels.
-   Avoid sensitive payload logging.
-   Keep traces useful rather than collecting everything blindly.
