# LearnGraph --- Events and Kafka

## 1. Purpose

Kafka connects long-running processing stages without tightly coupling
services.

Core pipeline:

``` text
Media
 ↓
Transcription
 ↓
Knowledge
 ↓
Search
 ↓
RAG
```

------------------------------------------------------------------------

# 2. Event Principles

Events should be:

-   Immutable.
-   Versioned.
-   Idempotently consumable.
-   Small enough to transport efficiently.
-   Rich enough to correlate with the originating workflow.
-   Explicit about ownership.

Do not put large video/audio content into Kafka.

Store files in MinIO and publish references.

------------------------------------------------------------------------

# 3. Event Envelope

``` json
{
  "eventId": "evt-123",
  "eventType": "TranscriptCompleted",
  "aggregateId": "VID-123",
  "occurredAt": "2026-09-25T10:00:00Z",
  "correlationId": "req-456",
  "producer": "transcription-worker",
  "schemaVersion": 1,
  "payload": {}
}
```

------------------------------------------------------------------------

# 4. Topics

Initial topic model:

``` text
media.video.uploaded
media.video.processing.requested
media.video.processing.completed
media.video.processing.failed

transcription.requested
transcription.completed
transcription.failed

knowledge.extraction.requested
knowledge.extraction.completed
knowledge.extraction.failed

search.indexing.requested
search.indexing.completed
search.indexing.failed
```

The exact topic list can evolve with the implementation.

------------------------------------------------------------------------

# 5. Partitioning

For video-processing events, a natural partition key is:

``` text
videoId
```

This helps preserve ordering for events belonging to the same video
within a partition.

Example:

``` text
video A → partition 1
video B → partition 3
video C → partition 2
```

This allows independent videos to process concurrently.

------------------------------------------------------------------------

# 6. Consumer Groups

Example:

``` text
transcription-workers
knowledge-workers
search-indexers
```

Each consumer group independently processes relevant events.

------------------------------------------------------------------------

# 7. Ordering

Kafka guarantees ordering within a partition, not globally.

Therefore business logic must not assume:

``` text
all system events are globally ordered
```

Instead, use:

``` text
aggregate ID
event version
state validation
```

------------------------------------------------------------------------

# 8. Idempotency

Every consumer should assume duplicate delivery.

Possible table:

``` text
processed_event
---------------
event_id
consumer_name
processed_at
```

Unique constraint:

``` text
(event_id, consumer_name)
```

Domain-specific idempotency may be preferable where processing naturally
has a unique business key.

------------------------------------------------------------------------

# 9. Outbox

Transaction:

``` text
BEGIN

UPDATE video
SET status = 'PROCESSING';

INSERT INTO outbox_event (...);

COMMIT;
```

Then:

``` text
Outbox Publisher
      ↓
Kafka
```

This avoids the dual-write problem.

------------------------------------------------------------------------

# 10. Retry

Transient:

``` text
Database timeout
Network timeout
Temporary provider failure
```

Retry with:

``` text
exponential backoff
jitter
maximum attempts
```

Permanent:

``` text
Invalid media
Invalid schema
Unsupported format
```

should not be retried forever.

------------------------------------------------------------------------

# 11. Dead Letter Queue

After retry exhaustion:

``` text
Main Topic
    ↓
Retry
    ↓
Retry
    ↓
DLQ
```

DLQ records should retain enough information for diagnosis and replay.

------------------------------------------------------------------------

# 12. Event Versioning

Example:

``` text
schemaVersion = 1
```

If payload changes incompatibly:

``` text
schemaVersion = 2
```

Consumers should have a controlled migration strategy.

------------------------------------------------------------------------

# 13. Event Replay

Because processing is asynchronous, a failed derived system can
potentially rebuild:

``` text
Search index
Embeddings
Concept projections
```

from durable source data/events.

Replay must be designed to be idempotent.

------------------------------------------------------------------------

# 14. Failure Example

``` text
Knowledge Worker
      ↓
PostgreSQL succeeds
      ↓
Kafka acknowledgement fails
      ↓
Event delivered again
```

The second delivery must not create duplicate relationships.

------------------------------------------------------------------------

# 15. Event Metadata

Always preserve:

``` text
eventId
correlationId
aggregateId
eventType
schemaVersion
occurredAt
producer
```

These fields are essential for observability and debugging.
