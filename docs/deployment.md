# LearnGraph --- Deployment

## 1. Deployment Evolution

``` text
Local
 ↓
Docker Compose
 ↓
CI/CD
 ↓
Container Registry
 ↓
Kubernetes
```

Kubernetes is a later deployment target, not a prerequisite for
development.

------------------------------------------------------------------------

# 2. Local Development

Docker Compose services:

``` text
PostgreSQL
Kafka
Redis
MinIO
Tempo
Prometheus
Loki
Grafana
```

Application services can run:

``` text
inside Docker
```

or locally from the IDE.

------------------------------------------------------------------------

# 3. Containerization

Each deployable application should have:

``` text
Dockerfile
Health endpoint
Configuration
Logging
Graceful shutdown
```

Use immutable image versions.

Avoid relying on:

``` text
latest
```

for production deployments.

------------------------------------------------------------------------

# 4. Configuration

Separate:

``` text
Application code
Configuration
Secrets
```

Example:

``` text
DATABASE_URL
KAFKA_BOOTSTRAP_SERVERS
REDIS_URL
MINIO_ENDPOINT
OTEL_EXPORTER_OTLP_ENDPOINT
```

Secrets should not be committed to Git.

------------------------------------------------------------------------

# 5. CI Pipeline

Example:

``` text
Push
 ↓
Compile
 ↓
Unit Tests
 ↓
Integration Tests
 ↓
Static Analysis
 ↓
Build Image
 ↓
Push to GHCR
```

------------------------------------------------------------------------

# 6. CD Pipeline

Later:

``` text
Image
 ↓
Deployment
 ↓
Smoke Tests
 ↓
Health Checks
 ↓
Rollout
```

------------------------------------------------------------------------

# 7. Kubernetes

Potential deployments:

``` text
gateway
auth-service
media-service
media-worker
transcription-worker
knowledge-worker
search-service
learning-service
rag-service
```

Scale workers independently.

------------------------------------------------------------------------

# 8. Health Checks

Expose:

``` text
/readiness
/liveness
```

Readiness answers:

> Can this instance receive traffic?

Liveness answers:

> Is this process still functioning?

Do not make liveness depend on every external service.

------------------------------------------------------------------------

# 9. Graceful Shutdown

Before shutdown:

``` text
Stop accepting new work
 ↓
Finish or safely checkpoint current work
 ↓
Commit offsets where appropriate
 ↓
Close resources
```

This is especially important for Kafka workers.

------------------------------------------------------------------------

# 10. Resource Management

Set:

``` text
CPU requests
CPU limits
Memory requests
Memory limits
```

CPU-heavy media workers should have different resource profiles from API
services.

------------------------------------------------------------------------

# 11. Autoscaling

Potential HPA signals:

``` text
CPU
Memory
Kafka lag
Request rate
```

Kafka lag may be more useful than CPU for event-processing workers.

------------------------------------------------------------------------

# 12. Deployment Security

Use:

``` text
Non-root containers
Minimal base images
Read-only filesystem where practical
Secret management
Network policies
TLS
Dependency scanning
```

------------------------------------------------------------------------

# 13. Production Rule

Do not deploy infrastructure merely to demonstrate that it exists.

Every component should have:

``` text
Reason
Configuration
Monitoring
Failure strategy
Recovery strategy
```
