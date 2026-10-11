# Architecture Decision Records

This file is the short-form decision index. The rationale and full product context live in the root [README](../README.md).

## ADR-001: Automotive as the flagship demo
**Decision:** Start with a simulated dashboard/feature-help experience.  
**Rationale:** It clearly demonstrates context, voice, retrieval, and uncertainty handling.  
**Constraint:** No real vehicle telemetry, diagnostics, or controls in the MVP.

## ADR-002: One answer engine for voice and text
**Decision:** Typed input and speech transcripts enter the same answer pipeline.  
**Rationale:** Prevents behavior and answer quality from diverging across modes.

## ADR-003: Push-to-talk, not always-on voice
**Decision:** Use an explicit microphone action for the first release.  
**Rationale:** Lower implementation and privacy complexity; clear user consent.

## ADR-004: RAG before fine-tuning
**Decision:** Ground answers in approved, versioned documents.  
**Rationale:** Documents can be updated, scoped, cited, and removed without model retraining.

## ADR-005: PostgreSQL + pgvector
**Decision:** Use one database for relational records and vector retrieval initially.  
**Rationale:** Less infrastructure for a portfolio MVP. Reassess against measured retrieval and scale needs.

## ADR-006: Modular monolith
**Decision:** Keep API, ingestion, answer engine, and speech as modules initially.  
**Rationale:** Clear boundaries without premature distributed-system overhead.

## ADR-007: Safe abstention
**Decision:** Clarify or abstain when evidence is insufficient or ambiguous.  
**Rationale:** A confident unsupported answer is worse than a transparent limitation.

## ADR-008: Tenant isolation is enforced in code
**Decision:** Scope every query by authorized project/tenant on the server.  
**Rationale:** Client-provided project IDs and origins are not authorization by themselves.

## ADR-009: No raw audio retention by default
**Decision:** Process audio without retaining it unless a future feature explicitly requires it and users are informed.  
**Rationale:** Data minimization and lower privacy risk.

## ADR-010: Evaluate before adding complexity
**Decision:** Maintain a small golden set covering answerable, unanswerable, ambiguous, stale, voice, and isolation cases.  
**Rationale:** Add reranking, extra providers, or agentic actions only when measurements justify them.
