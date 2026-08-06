# ADR 0002 — Technology Stack

**Status:** ACCEPTED (2026-08-06, Phase 0 approval)

## Context

StockForge must be production-style (realistic, testable, deployable) yet learnable in 30-minute units by someone new to development. It must also support a later HFT-evolution phase that involves JVM-style performance analysis: garbage collection, lock contention, object pooling, profiling, and nanosecond/microsecond measurement.

## Decision

| Layer | Choice | Alternatives considered |
|---|---|---|
| Backend | Java 21 LTS + Spring Boot 3.x + Maven | Go, Rust, C#/.NET |
| Frontend | React 18 + TypeScript + Vite | Vue, Svelte |
| Realtime | WebSocket | SSE, polling |
| Database | PostgreSQL 16 | MySQL |
| Cache | Redis 7 | Memcached |
| Messaging | Apache Kafka 3.x (KRaft) | Redpanda, RabbitMQ |
| Container | Docker + Docker Compose | Podman |
| Kubernetes | kind locally → EKS later | minikube, k3s |
| CI/CD | GitHub Actions + Jenkins (both first-class) | GitLab CI, CircleCI |
| Performance testing | k6 (primary); JMeter/Gatling as alternatives | Locust, vegeta |
| Observability | Prometheus + Grafana + OpenTelemetry + Loki; New Relic later | Datadog |
| IaC | Terraform + Helm | Pulumi, Ansible |
| Perf analysis | JFR/JMC, async-profiler, GC logs | VisualVM, MAT |

## Rationale (key decisions)

- **Java/Spring Boot:** the trading industry's dominant stack; the JVM gives us the exact performance-engineering material for the HFT phase (GC tuning, JFR, async-profiler, object pooling, lock analysis). It is heavier to learn than Go, but that cost is repaid by the HFT curriculum.
- **PostgreSQL + Redis + Kafka:** the canonical trading data stack; each teaches a distinct distributed-systems concept (transactions/locks, caching, event streaming).
- **Both CI/CD systems:** the core learning objective is *how enterprises implement CI/CD and make performance part of the pipeline* — not Jenkins syntax alone.

## Alternatives (captured)

- Go backend: simpler concurrency and lower allocation overhead, but drops the JVM/HFT performance curriculum. If chosen, we lose GC/JFR lessons and would need a different perf-analysis track.
- JMeter/Gatling instead of k6: viable; we will compare at least once during Phase 16.
- Datadog over Prometheus/Grafana: better UX but paid; Prometheus/Grafana run free locally and teach the concepts.

## Consequences

- Positive: realistic production stack; strong HFT-phase alignment; large ecosystem of learning resources.
- Negative: heavier initial learning curve (JVM, Maven, Spring); mitigated by the teach-before-automating rule and start-small service design.
- Decision can be revisited if the user prefers Go; treat the backend language as a required approval point.
