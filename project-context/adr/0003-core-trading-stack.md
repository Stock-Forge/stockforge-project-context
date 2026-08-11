# ADR 0003 — Core Trading Stack (Java-first trading engine)

**Status:** PROPOSED (2026-08-10) — awaiting approval before becoming ACCEPTED.

## Context

StockForge will eventually include the real-time trading heart: market data ingestion, order
matching, strategy, risk, and order management. This is the **hot path** — the part where
microseconds matter and where the HFT-evolution phase (PROJECT_CONTEXT §17 / Phase 26) will live.
A study session on GC, JFR, and lock contention (see LEARNING_LOG bonus entry, 2026-08-10) made the
architectural split explicit: the platform/control-plane concerns (REST, admin, configuration,
monitoring, reporting) and the latency-critical trading path are *different engineering problems* and
should not be forced into one stack.

## Decision

1. **Core trading engine = plain Java (no Spring Boot).** The first implementation of the
   latency-critical path — market data handling, the trading engine, strategy, risk, order
   management, and the exchange gateway — is built as a lean Java application using the JVM directly
   (threads, synchronisation, arrays, etc.), NOT a Spring Boot service.
2. **Spring Boot remains the control plane.** REST APIs, admin, configuration, monitoring and
   reporting services (like `stockforge-api`, `stockforge-auth`) stay on Spring Boot. Databases,
   Kafka, Redis, etc. stay off the hot path.
3. **Rationale: learning objective first.** The decision is Java-first so we *deeply learn JVM
   performance engineering*: GC, JFR, JIT, threads, locks, lock-free programming, CPU/memory
   optimization, and µs/ns measurement. This is NOT a claim that Java is universally the best HFT
   language.
4. **C++ comparison later.** Once the Java engine is measured and understood, an equivalent C++
   implementation can be built for comparison: latency, GC vs manual memory management, CPU usage,
   memory behavior, locking/concurrency, networking, and p99/p99.9/p99.99.

## Alternatives

- Build the trading core as another Spring Boot microservice: easier to start, but drags the
  control-plane framework onto the hot path and dilutes the JVM-performance curriculum.
- Start with C++: closer to real low-latency practice, but skips the JVM learning track that this
  project was set up to teach (ADR 0002 rationale).

## Consequences

- Positive: a clear, measurable JVM performance-engineering track; a well-defined boundary between
  control plane and hot path; a C++ comparison target for later phases.
- Negative: an extra build shape in the project (plain Java vs Spring Boot); the trading core must
  solve its own wiring/HTTP/questions that Boot would have handled — acceptable because the hot path
  intentionally avoids that machinery.

## Links

- PROJECT_CONTEXT.md §17 (HFT Evolution), §18 phases (6 matching engine, 7 market data)
- LEARNING_LOG bonus entry (2026-08-10) — GC / JFR / lock contention
- ADR 0002 (technology stack) — the JVM performance curriculum it promised is realized here
