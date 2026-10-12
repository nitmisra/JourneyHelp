# JourneyHelp Product Requirements Document

**Version:** 0.1 · **Date:** October 2026 · **Development method:** Specification-Driven Development (SDD)

## 1. Product definition

JourneyHelp is an embeddable, context-aware product assistant that helps users understand and use a product without leaving their current journey. The first release is a simulated automotive dashboard: users select an indicator, ask a text question, and receive an answer grounded in illustrative sample-manual content, with source references.

### Problem
Users often leave a product to search manuals or help articles and struggle to identify information relevant to their current screen, product, or model. JourneyHelp brings help into the product context.

### Goals
1. Demonstrate a complete browser-widget → API → retrieval → answer flow.
2. Prove that host-page context improves answer relevance.
3. Return application-owned source references.
4. Keep the widget independent of the host UI framework.
5. Establish a foundation for voice, ingestion, and additional verticals.

### Non-goals
Real vehicle diagnostics/telemetry/control; banking account access or transactions; autonomous actions; production authentication or multi-tenancy; vector search and agent orchestration; voice input/output in the first slice.

## 2. Personas and user stories

- **End user:** wants help about the product or feature currently on screen.
- **Demo developer:** wants a local, reproducible end-to-end demo.
- **Integrating developer:** wants to pass context to an embeddable widget.

| ID | User story | Priority |
|---|---|---|
| US-01 | Select a dashboard item and ask what it means. | P0 |
| US-02 | Receive an answer relevant to the selected item. | P0 |
| US-03 | See source references for an answer. | P0 |
| US-04 | Receive an honest fallback for unsupported questions. | P0 |
| US-05 | Embed the widget without adopting its UI framework. | P0 |
| US-06 | Supply product and page context. | P0 |
| US-07 | Run without an LLM key. | P0 |
| US-08 | Configure a server-side LLM key. | P0 |
| US-09 | Ask follow-up questions in one session. | P1 |
| US-10 | Ask by voice and choose spoken or text responses. | P2 |
| US-11 | Ingest, version, and publish documentation. | P2 |
| US-12 | Evaluate quality and review analytics. | P2 |

## 3. Functional requirements

### FR-01 — Automotive demo context
The demo MUST display four simulated items: tire pressure, engine indicator, rear defogger, and oil pressure.

- **AC-01.1:** All four items are visible.
- **AC-01.2:** Exactly one item is selected at initialization.
- **AC-01.3:** Selecting an item updates visual state without a page reload.
- **AC-01.4:** Each request includes product, model year, page, locale, and current selected item.
- **AC-01.5:** The demo says it is simulated and not connected to a real vehicle.

### FR-02 — Embeddable widget
The widget MUST be a TypeScript Web Component named `<journey-help-widget>`.

- **AC-02.1:** Text input and submit action are available.
- **AC-02.2:** User messages, answers, loading state, and recoverable errors are displayed.
- **AC-02.3:** Duplicate submissions are prevented while a request is pending.
- **AC-02.4:** User/model text is rendered as text, never executable HTML.
- **AC-02.5:** Controls have accessible labels and are keyboard operable.
- **AC-02.6:** Layout adapts to narrow screens.

### FR-03 — Chat API
Expose `GET /health` and `POST /v1/chat`.

- **AC-03.1:** Requests use typed validation schemas.
- **AC-03.2:** Invalid or blank messages are rejected.
- **AC-03.3:** Context reaches retrieval and answer generation.
- **AC-03.4:** Response includes status, answer, sources, response mode, demo flag, and model label.
- **AC-03.5:** Provider failures return controlled errors without secrets or stack traces.
- **AC-03.6:** Health returns HTTP 200 while the service is running.

### FR-04 — Knowledge retrieval
Retrieve relevant records from `examples/automotive/demo_knowledge.json`.

- **AC-04.1:** Records include stable ID, title, section, content, keywords, and product applicability.
- **AC-04.2:** Ranking uses question terms and selected-item context.
- **AC-04.3:** Return at most three records.
- **AC-04.4:** Defined evaluation questions retrieve the expected record.
- **AC-04.5:** Unsupported ordinary questions are marked `insufficient_evidence`.
- **AC-04.6:** Source references are built from retrieved records, not model-generated IDs.

Use keyword retrieval in the first slice. Defer embeddings and pgvector until a baseline is evaluated.

### FR-05 — LLM integration
Support one configurable OpenAI integration on the backend.

- **AC-05.1:** Read `OPENAI_API_KEY` and `OPENAI_MODEL` from server-side environment configuration.
- **AC-05.2:** Provider credentials never appear in browser code or API responses.
- **AC-05.3:** Prompt includes question, context, and retrieved evidence.
- **AC-05.4:** Model is instructed to ground factual claims and acknowledge missing evidence.
- **AC-05.5:** Safety-sensitive questions receive cautious guidance.
- **AC-05.6:** Provider failure returns a controlled error.
- **AC-05.7:** Tests do not require external provider calls.

Without a key, use a clearly labelled deterministic local responder. Do not describe it as an LLM response.

### FR-06 — Sources and status
Statuses are `answered`, `needs_clarification`, and `insufficient_evidence`. Each source has `id`, `title`, and `section`. The widget displays returned sources. The system never fabricates source IDs or presents unsupported answers as verified guidance.

### FR-07 — Automotive safety
JourneyHelp MUST NOT claim to diagnose a real vehicle.

- Red oil-pressure, brake, steering, smoke, and overheating reports receive cautious responses.
- Responses prioritize safe stopping where appropriate and consulting the official manual or qualified assistance.
- No vehicle commands or control actions exist.
- Safety behavior is tested independently of retrieval.

### FR-08 — Local developer experience
Python dependencies and commands are documented; API and frontend run locally; demo works without an API key; `.env.example` contains placeholders only; tests do not make paid provider calls.

### FR-09 — Future voice capability (deferred)
When implemented, voice MUST use the same answer engine as text and support push-to-talk, transcription, voice/text preference, transcript display, stop/replay/switch controls, and typed fallback.

## 4. API contract

### Request

```json
{
  "project_id": "automotive-demo",
  "session_id": "demo-session",
  "message": {"type": "text", "content": "What does this warning mean?"},
  "context": {"product":"Journey Motors Atlas", "model_year":"2025", "page":"/dashboard", "selected_item":"tire-pressure", "locale":"en-IN"},
  "response_preferences": {"mode":"text"}
}
```

### Response

```json
{
  "status":"answered",
  "answer":"A tire-pressure warning can indicate that one or more tires may be under-inflated.",
  "sources":[{"id":"tire-pressure","title":"Tire pressure warning","section":"Dashboard indicators"}],
  "response_mode":"text",
  "needs_mode_selection":false,
  "demo_mode":true,
  "model":"local-demo-responder"
}
```

Message content must be 1–4,000 characters. Response mode accepts `ask`, `text`, or `voice`; voice delivery is not implemented in this release.

| HTTP | Meaning | Client behavior |
|---|---|---|
| 200 | Request handled | Display answer and sources |
| 422 | Invalid request | Show validation feedback |
| 502 | Configured provider failed | Show recoverable error |
| 500 | Unexpected failure | Show generic error; log details server-side |

## 5. Decision rules

1. Context is a relevance signal, not proof of diagnosis.
2. Evidence takes precedence over fluent-sounding language.
3. Backend owns source IDs and labels.
4. Server configuration controls LLM usage.
5. Critical safety behavior must not rely only on prompt instructions.
6. No autonomous vehicle actions are permitted.

## 6. Non-functional requirements

| ID | Requirement | Target |
|---|---|---|
| NFR-01 | Local setup | Start API and demo using documented commands |
| NFR-02 | Latency | Measure baseline; target p95 < 5 seconds for normal local requests, excluding unusually slow provider responses |
| NFR-03 | Secret handling | No provider keys in frontend or responses |
| NFR-04 | Input validation | Validate required fields and message length |
| NFR-05 | UI resilience | Loading and recoverable error states |
| NFR-06 | Accessibility | Keyboard controls, labels, contrast, announced updates |
| NFR-07 | Testability | Core tests run without paid external calls |
| NFR-08 | Observability | Log failures and mode metadata without secrets |
| NFR-09 | Reproducibility | Document dependencies and configuration |
| NFR-10 | Honest claims | No unmeasured quality or production-readiness claims |

Latency is a target to measure, not an existing performance claim.

## 7. SDD workflow

For every feature: (1) review product requirement; (2) write/update feature specification; (3) define scenarios and acceptance criteria; (4) update API/data contracts; (5) write tests from criteria; (6) implement the smallest conforming change; (7) run tests and validate demo; (8) update spec and decision log if behavior changes.

The PRD is the product-level source of truth; feature specs are implementation contracts; tests provide executable verification. See `specs/README.md`.

## 8. Test strategy

- **Unit:** retrieval ranking, validation, source construction, context, safety matching.
- **Integration:** API → retrieval → deterministic responder; mocked provider success/failure.
- **End-to-end:** select item → ask → see answer and source → change item → ask again.

Minimum tests: health; tire pressure; rear defogger; unsupported question; oil-pressure safety; blank message rejection; no-key mode; configured LLM path; provider failure; source integrity; context update; HTML injection handling.

## 9. Delivery plan

1. Foundation: repository, schemas, health endpoint, test harness.
2. Knowledge: sample content, retrieval, source construction, safety rules, local responder.
3. LLM: OpenAI adapter, server configuration, controlled errors, mocked tests.
4. Widget: Web Component, request handling, loading/error states, source rendering.
5. Demo: dashboard, selection, context propagation, responsive layout.
6. Validation: run P0 tests and manually verify acceptance criteria.

## 10. Decisions and rationale

| Decision | Rationale |
|---|---|
| Automotive first | Concrete context-aware demo |
| Simulated dashboard | Avoids real-vehicle dependencies and risks |
| Text first | Proves core loop before speech complexity |
| Web Component | Framework-independent embedding |
| FastAPI | Typed Python API and fast iteration |
| One optional OpenAI provider | Real LLM path without premature abstraction |
| Keyword retrieval | Measurable baseline before vector search |
| Local responder | Reproducible demo without credentials |
| Backend-owned sources | Avoids invented source IDs |
| Modular monolith | Easier to develop and debug |

## 11. Release checklist

- [ ] API and frontend start using documented commands.
- [ ] Four dashboard items can be selected.
- [ ] Current selection reaches the API.
- [ ] Answers include relevant sources.
- [ ] Local mode works without API key.
- [ ] Configured LLM path works when available.
- [ ] Provider errors do not leak secrets.
- [ ] Unsupported questions are handled honestly.
- [ ] Critical questions receive cautious guidance.
- [ ] Widget handles loading and recoverable errors.
- [ ] All P0 automated tests pass.

## 12. Definition of done

A feature is done only when its specification and acceptance criteria exist, contracts are current, tests cover normal and relevant edge cases, the demo verifies behavior end to end, security/safety implications are reviewed, and documentation reflects what is implemented. Code generation or successful startup alone is insufficient.
