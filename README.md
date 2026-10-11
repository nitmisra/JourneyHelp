# JourneyHelp
### A context-aware, voice-first help layer for digital products

JourneyHelp is an open-source portfolio project exploring how any product can offer contextual help through **voice or text**, grounded in approved product knowledge.

The flagship demo is an automotive experience: a driver asks about a dashboard indicator or vehicle feature, and JourneyHelp uses the vehicle context plus the owner's manual to provide a concise, source-grounded explanation. The same answer engine should also work for banking and B2B SaaS demonstrations.

> **Project status:** Design and starter-repository stage. This README records the intended product and technical decisions; it does not claim that the proposed APIs or integrations are already implemented.

---

## 1. Product vision

**Make complex products easier to understand at the moment a person needs help.**

Most help centers make users search documentation, leave their current workflow, or translate product terminology into a support ticket. JourneyHelp aims to put a small, embeddable assistant inside the product journey itself.

A host product supplies relevant context—such as the page being viewed, product model, feature, locale, or selected warning—and JourneyHelp combines that context with approved knowledge to answer the user's question.

### The core promise

**Any product can provide context and knowledge to the same help engine; users can interact through voice or text.**

JourneyHelp is not intended to be a generic chatbot that confidently answers everything. It should answer from evidence, ask a clarifying question when needed, and say when the available documentation is insufficient.

### Who might use it?

- **Product teams** that want to reduce friction in complex digital journeys.
- **Customer experience and support teams** that want to resolve routine questions in context.
- **OEMs and automotive digital teams** that need to explain vehicle features and documentation.
- **B2B SaaS vendors** whose users struggle with configuration and workflows.
- **Banks and financial-service providers** that need clear explanations of approved product terms and benefits.

---

## 2. Why automotive is the flagship demo

Automotive is the most memorable demonstration of the product idea because it naturally combines contextual knowledge, voice interaction, and a real-world need for concise guidance.

Example:
1. A user opens a simulated dashboard for a specific vehicle model and year.
2. The user selects a dashboard indicator or feature.
3. The user asks, “What does this mean?” by voice or text.
4. JourneyHelp uses the supplied vehicle context and relevant owner's-manual content.
5. It answers with a source citation, asks for more detail if the symbol is ambiguous, and gives appropriately cautious guidance.

The first version will **simulate the vehicle interface**. It will not connect to an actual car, read live vehicle telemetry, diagnose a fault, or control vehicle systems. Those capabilities would require validated data, integrations, safety engineering, and partner access beyond the MVP.

### Why not start with banking or SaaS?

| Candidate | Strength | Trade-off | Role in this project |
|---|---|---|---|
| Automotive | Memorable voice + context + manual-based RAG demo | Safety-sensitive; hardware/OEM integration can be difficult | Flagship showcase |
| B2B SaaS | Straightforward web demo and plausible early pilot | Crowded category; less visually distinctive | Reuse the same engine to prove portability |
| Banking | Clear support use cases and measurable service value | Privacy, account authorization, and financial accuracy add complexity | Later example using public product/benefit information only |

This is a portfolio and learning choice, not a claim that automotive is the easiest route to revenue. For a first paid pilot, a narrowly scoped SaaS use case may be simpler to validate.

---

## 3. MVP scope

### In scope

- An embeddable help widget with text input.
- Push-to-talk voice input (not always-on listening).
- A first-run choice between **spoken answer** and **text answer**.
- A session-level preference that users can change at any time.
- Visible transcript for spoken answers.
- Stop, replay, and switch-mode controls.
- A single shared answer engine for voice and text.
- Retrieval-augmented generation (RAG) over approved documents.
- Source citations and an explicit “I don't have enough evidence” behavior.
- Host-provided context, such as product, model/year, page, selected feature, and locale.
- A simulated automotive dashboard and an owner's-manual knowledge set.
- A second, small SaaS demo using the same engine.
- Basic evaluation cases, structured logs, and a local development setup.
- Clear setup instructions and architecture documentation.

### Out of scope for the MVP

- Always-on microphone access or wake-word detection.
- Real vehicle telemetry, OBD/CAN integration, or control of vehicle functions.
- Autonomous actions that affect a customer's account, vehicle, or finances.
- Access to private banking account data.
- Multi-agent orchestration for its own sake.
- Fine-tuning a foundation model.
- Training a custom speech-recognition or speech-synthesis model.
- Enterprise SSO, complex RBAC, billing, and multi-region deployment.
- Guaranteed support for every browser, language, and device.

**MVP success criterion:** a user can ask a contextual question in the automotive demo, receive a grounded answer with source evidence, choose voice or text, and see the same core answer behavior in the SaaS demo.

---

## 4. User experience decisions

### Voice and text

The interaction should respect the user's choice rather than assume that voice is always appropriate.

1. The user can type or press a microphone button.
2. The first time an answer is ready, the assistant asks whether the user wants **voice or text** if no preference has been selected.
3. The choice is remembered for the current session.
4. Spoken answers also display their transcript.
5. Text answers do not automatically play audio.
6. The user can stop speech, replay it, or switch modes.
7. If browser speech support is unavailable or permission is denied, text remains usable.

### Why push-to-talk first?

Push-to-talk is easier to implement, test, explain, and use with clear consent than always-on listening. It reduces accidental recording and avoids the complexity of wake-word detection and continuous microphone access.

### Speech implementation decision

Start with browser speech APIs behind a small adapter interface. This keeps the first demo inexpensive and avoids making a cloud speech provider a prerequisite. Browser support and recognition quality vary, so the UI must detect capability and provide a text fallback.

Keep speech recognition and speech synthesis replaceable. A hosted STT/TTS provider or real-time speech API can be added later if evaluation shows that browser speech is insufficient. Do not store raw audio by default.

### One answer engine

Typed questions and transcribed speech must enter the same question-processing, retrieval, answer-generation, and evidence-validation path. Separate voice and text answer pipelines would drift in quality and behavior.

---

## 5. Architecture decisions and why

### High-level architecture

```text
Host product / demo page
        |
        | context + user question
        v
Embeddable widget / SDK
  - text input
  - push-to-talk
  - mode selection
  - transcript / playback controls
        |
        v
API boundary
  - project and origin checks
  - rate limits
  - request validation
  - tenant/project scoping
        |
        v
Shared answer engine
  1. Normalize question and context
  2. Retrieve approved evidence
  3. Check evidence sufficiency
  4. Generate grounded answer
  5. Validate citations and safety policy
        |
        +------> Knowledge ingestion and index
        |         manuals / FAQs / approved docs
        |
        +------> LLM provider adapter
        |
        +------> Embedding provider adapter
        |
        +------> Speech adapter (STT / TTS)
        |
        v
Answer + sources + status + selected response mode
        |
        v
Widget displays text or plays speech (with transcript)
```

### Technology choices

| Area | Initial decision | Why | Revisit when |
|---|---|---|---|
| Widget | TypeScript Web Component | Framework-neutral embedding target; can work in React, Vue, or plain HTML hosts | Integration tests show a different packaging strategy is materially simpler |
| Demo UI | React + Vite + TypeScript | Fast to build a polished interactive demo and share components | Demo complexity justifies a different framework |
| API | Python + FastAPI | Simple API development, strong ecosystem for retrieval and evaluation | Measured performance or deployment needs require a change |
| Data store | PostgreSQL | Durable relational data, metadata filters, sessions, sources, and project configuration in one familiar store | Scale or isolation requirements demonstrate a need |
| Vector search | pgvector in PostgreSQL | Avoid a separate vector database during the MVP; keep source metadata and vectors close together | Retrieval quality, volume, or operational load warrants a dedicated service |
| LLM | One provider behind an adapter | Ship a complete vertical slice before spending time on provider abstraction; retain an exit path | Quality, latency, data policy, or cost makes a second provider useful |
| Embeddings | One embedding model behind an adapter | Keep indexing reproducible and simple initially | Evaluation shows a different model materially improves retrieval |
| Speech | Browser STT/TTS behind adapters | Low setup friction for a portfolio demo and no mandatory speech bill | Cross-browser tests or voice quality fail target requirements |
| Local services | Docker Compose | Reproducible local setup for API and database | Deployment target needs a different orchestration model |
| Tests | Python tests + TypeScript tests | Test answer logic, API contracts, widget states, and regressions independently | A shared test strategy becomes necessary |
| CI | GitHub Actions | Run linting and tests on every pull request | Repository needs additional release/security workflows |

These are starting decisions, not permanent commitments. Prefer the simplest design that lets the project demonstrate its core promise and measure quality.

### Why RAG instead of fine-tuning?

Product manuals and FAQs change. RAG allows the project to update the knowledge corpus, cite the evidence used, filter by product/version, and remove or replace stale content without retraining a model. Fine-tuning is not needed to prove the MVP.

### Why PostgreSQL + pgvector instead of a separate vector database?

At portfolio scale, a separate vector service adds configuration, deployment, and operational work before the product needs it. PostgreSQL can hold document metadata, access scope, session records, and embeddings together. The repository should hide vector-store operations behind an interface so a later migration remains possible.

### Why one LLM provider initially?

The goal is to learn and demonstrate the complete product lifecycle, not to build a model benchmark platform first. A small provider interface avoids hard-coupling the app while keeping the initial implementation focused. Provider portability must not become an excuse to delay a working end-to-end demo.

### What should use an LLM—and what should not?

- **Use deterministic code** for project scoping, origin checks, access control, mode preferences, rate limits, and allowed actions.
- **Use retrieval** to find evidence from approved documentation.
- **Use the LLM** to interpret the user's question and compose a concise answer from retrieved evidence.
- **Use explicit validation** to check whether sources exist and whether the response meets policy.
- **Do not let the model alone authorize sensitive actions** or decide whether a user can access data.

This keeps the product more predictable and makes failures easier to test.

---

## 6. Knowledge ingestion and RAG decisions

The knowledge lifecycle is:

1. Accept an approved URL, PDF, Markdown, or plain-text source.
2. Parse and normalize the content.
3. Split it into meaningful chunks without losing important headings or nearby context.
4. Attach metadata: tenant/project, product/model, locale, document version, source, effective date, and status.
5. Generate embeddings and index the chunks.
6. Run retrieval tests and inspect source coverage.
7. Publish a version; retain the ability to disable or roll back a source.

### Retrieval policy

- Filter by project/tenant and relevant product context **before** exposing evidence to the answer generator.
- Combine semantic retrieval with keyword matching where practical.
- Preserve source titles, URLs or document references, headings, and version information.
- Return citations with answers.
- If evidence is weak, conflicting, outdated, or missing, ask a clarifying question or abstain rather than invent an answer.
- Treat retrieved document content as untrusted input, not as instructions to the model.
- Never treat a document's existence as proof that it is current or authoritative.

### Chunking and ranking

Start with heading-aware chunks and a configurable overlap. Use a small top-k retrieval set and log which chunks were selected. Add reranking only if evaluation reveals a meaningful retrieval problem; do not add it merely because it is a popular architecture component.

### Automotive safety policy

The MVP explains documentation; it does not diagnose a vehicle or replace professional assistance. For safety-critical warnings, ambiguous symbols, braking/steering issues, smoke, overheating, or other urgent symptoms, the assistant should avoid speculative troubleshooting and direct the user to the vehicle's official instructions and appropriate professional or emergency assistance. Specific warning behavior must be reviewed against the chosen vehicle documentation.

---

## 7. API and integration contract (proposed)

The API names below are design targets. They should be treated as proposals until implemented and tested.

### Initial endpoints

| Endpoint | Purpose |
|---|---|
| `POST /v1/chat` | Submit a typed or transcribed question with context; return an answer, sources, and mode state |
| `POST /v1/speech/transcriptions` | Optional server-side speech-to-text adapter endpoint |
| `POST /v1/speech/synthesis` | Optional server-side text-to-speech adapter endpoint |
| `POST /v1/projects` | Create a project through an authenticated admin interface |
| `GET /v1/projects/{project_id}/widget-config` | Fetch safe, public widget configuration |
| `POST /v1/knowledge/sources` | Register a knowledge source through an authenticated admin interface |
| `GET /v1/knowledge/jobs/{job_id}` | Check ingestion status |
| `POST /v1/feedback` | Capture helpful/not-helpful feedback |
| `GET /v1/analytics/summary` | Return scoped, aggregate usage and quality metrics |

### Example chat request

```json
{
  "project_id": "public-demo-project",
  "session_id": "session-123",
  "message": {
    "type": "text",
    "content": "What does this warning mean?"
  },
  "context": {
    "product": "Example vehicle",
    "model_year": "2025",
    "page": "/dashboard",
    "selected_item": "amber-engine-indicator",
    "locale": "en-IN"
  },
  "response_preferences": {
    "mode": "ask"
  }
}
```

### Example response shape

```json
{
  "status": "answered",
  "answer": "The selected indicator is described in the vehicle manual as ...",
  "sources": [
    {
      "title": "Owner's Manual",
      "section": "Dashboard indicators",
      "document_version": "demo-v1"
    }
  ],
  "response_mode": "text",
  "needs_mode_selection": true,
  "follow_up_suggestions": [
    "Show the exact symbol",
    "Explain the recommended next step"
  ]
}
```

These are illustrative contracts. The implementation should define typed request/response schemas and return explicit states for `answered`, `needs_clarification`, `insufficient_evidence`, and `error`.

### Embedding strategy

The target integration should eventually be a small script tag that loads the widget and receives a public project identifier plus optional context. Keep this as a goal—not a promise—until the SDK, origin checks, and versioning are implemented.

A public project ID is **not a secret**. Never place an LLM key, database credential, or privileged admin token in browser code. Origin allowlists help control embedding but do not replace authentication or server-side authorization.

---

## 8. Security, privacy, and trust decisions

Security boundaries belong in the first implementation, not as a later cleanup.

- Enforce tenant/project scoping on every knowledge query and data operation.
- Keep provider keys and database credentials server-side.
- Use authenticated admin routes for project and knowledge management.
- Validate request shape, size, locale, context fields, and allowed origins.
- Add rate limits and sensible request timeouts.
- Treat the host-provided context and retrieved documents as untrusted input.
- Do not allow arbitrary model-generated SQL, shell commands, URLs, or tool calls.
- Do not store raw audio by default; make retention behavior explicit.
- Avoid collecting personal or account data that is not needed for the demo.
- Define deletion and retention behavior for sessions, messages, and telemetry.
- Keep audit-friendly traces of retrieved source IDs and answer status, while avoiding unnecessary sensitive content in logs.
- Use synthetic/demo data only in the public repository.

For a later banking demo, do not retrieve private account information merely because the user asks. Account-specific answers require a trusted identity, authorization, consent, and a properly scoped server-side integration. The MVP banking example should use public card terms and benefit documentation only.

---

## 9. Evaluation: how we know it works

A convincing demo is not enough. Build a small, repeatable evaluation set before adding complex features.

Create 50–100 synthetic questions across automotive and SaaS, including:

- Questions directly answered by one document.
- Questions requiring multiple passages.
- Ambiguous questions that should trigger clarification.
- Unanswerable questions that should be refused or deferred.
- Conflicting or outdated documentation.
- Similar product models with different answers.
- Prompt-injection text inside a retrieved document.
- Attempts to access another project's knowledge.
- Voice transcription errors and unsupported browser capabilities.
- Mode changes, stop, replay, and transcript visibility.

Track at least:

| Metric | What it tells us |
|---|---|
| Retrieval hit rate / source relevance | Did retrieval find useful evidence? |
| Grounded-answer rate | Are factual claims supported by sources? |
| Correct abstention rate | Does the system avoid answering when evidence is inadequate? |
| Citation validity | Do citations point to retrieved, relevant sources? |
| Context-use accuracy | Did product/model/page context affect retrieval correctly? |
| Tenant-isolation tests | Can one project ever receive another project's content? |
| Voice task completion | Can users ask, hear/see, stop, replay, and change modes? |
| Latency and cost per answer | Is the experience practical to operate? |

Do not claim a quality percentage until the test set has been run and the measurement method is documented.

---

## 10. Proposed repository structure

```text
journeyhelp/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── .gitignore
├── .env.example
├── docker-compose.yml
├── apps/
│   ├── demo/                 # Automotive and SaaS demo experiences
│   └── admin/                # Later: knowledge and project management UI
├── packages/
│   ├── widget/               # Embeddable UI
│   ├── sdk/                  # Host integration helpers
│   └── contracts/            # Shared API types/schemas
├── services/
│   ├── api/                  # FastAPI routes and auth boundary
│   ├── answer_engine/        # Retrieval, answer generation, validation
│   ├── ingestion/            # Parsing, chunking, indexing
│   ├── speech/               # STT/TTS provider adapters
│   └── analytics/            # Events and aggregate metrics
├── connectors/               # Document and product-context connectors
├── examples/
│   ├── automotive/
│   ├── banking/
│   └── saas/
├── evals/                    # Golden questions and scoring scripts
├── tests/
│   ├── unit/
│   ├── integration/
│   └── security/
├── docs/
│   ├── architecture.md
│   ├── decisions.md
│   ├── threat-model.md
│   └── evaluation.md
└── .github/
    └── workflows/
        └── ci.yml
```

This structure separates the host-facing widget, backend API, answer logic, ingestion, and evaluation. It is intentionally modular without starting with microservices that must be deployed independently. The first implementation can run as one API service with internal modules; split deployment units only when there is a measured reason.

---

## 11. Delivery plan

An eight-week plan is a planning aid, not a deadline promise.

| Week | Goal | Exit criterion |
|---|---|---|
| 1 | Repository, local environment, widget shell, API health check | Demo page calls the local API successfully |
| 2 | LLM adapter and basic answer path | A typed question returns a structured answer |
| 3 | Document ingestion and RAG | Answers use approved chunks and include citations |
| 4 | Context-aware retrieval | Vehicle model/year and selected indicator influence results |
| 5 | Voice UX | Push-to-talk, transcript, voice/text selection, stop/replay work with fallback |
| 6 | Automotive flagship demo + small SaaS demo | Same answer engine serves both examples |
| 7 | Evaluation, security tests, logs | Golden set and tenant-isolation tests run repeatably |
| 8 | Documentation, packaging, CI, recorded demo | Another developer can run the project from the README |

### First coding milestone

Build the smallest end-to-end vertical slice:

1. Create the repository and environment template.
2. Implement a simple TypeScript chat widget.
3. Implement a FastAPI `POST /v1/chat` endpoint with typed request/response models.
4. Add one LLM provider adapter.
5. Return an answer with a placeholder source structure.
6. Add one test for the API contract.
7. Only then add document ingestion and RAG.

This sequence proves the integration path early. The placeholder source must be clearly marked as such and removed when real retrieval is implemented; never display fabricated citations as real evidence.

---

## 12. Commercial hypothesis

JourneyHelp could eventually be offered in three forms:

1. **Open-source core:** widget, local demo, documented API, evaluation examples.
2. **Hosted pilot:** managed knowledge ingestion, analytics, and simple onboarding for a design partner.
3. **Enterprise offering:** SSO, audit logs, RBAC, governed connectors, retention controls, and deployment choices.

The commercial hypothesis is that teams may value an embeddable, context-aware, source-grounded assistant more than a standalone chatbot. That hypothesis must be tested through user interviews and pilots; this README does not claim product-market fit.

### Potential initial customers

- B2B SaaS companies with complex setup and configuration flows.
- Automotive digital experience teams with manuals and feature guides.
- Customer support/product teams seeking to reduce repetitive “how do I?” questions.

Start by validating one narrow workflow with users. Do not build enterprise infrastructure before evidence of demand.

---

## 13. Known limitations and risks

- Browser speech recognition is inconsistent across browsers, operating systems, and languages.
- Retrieval quality depends on source quality, parsing, chunking, metadata, and evaluation.
- LLM output can be incorrect even when relevant evidence is retrieved.
- Automotive instructions can be safety-sensitive; source citations do not automatically make advice safe.
- Public demo content is not suitable for real customer or account data.
- The first version will not include production-grade multi-tenant certification, enterprise identity integrations, or a formal safety case.

The project should communicate uncertainty clearly and prefer safe, useful abstention over false confidence.

---

## 14. Decision log

| Decision | Chosen approach | Why |
|---|---|---|
| Flagship use case | Automotive dashboard/feature helper | Memorable demonstration of context + voice + product knowledge |
| Secondary proof | B2B SaaS help flow | Shows the engine is reusable and offers a plausible early pilot |
| MVP interaction | Text + push-to-talk voice | Demonstrates multimodality without always-on listening complexity |
| Voice preference | Ask once, remember for session, allow switching | Respects user preference and keeps behavior predictable |
| Spoken answer | Always show transcript | Accessibility, verification, and use in noisy environments |
| Text answer | Do not auto-play audio | Avoids surprising users and supports quiet/private use |
| Knowledge approach | RAG over approved documents | Freshness, citations, and source-level updates without model retraining |
| Initial vector store | PostgreSQL + pgvector | Fewer services and simpler local development |
| Initial model strategy | One LLM + one embedding model behind adapters | Build the full product first while preserving an exit path |
| Answer logic | One shared pipeline for voice and text | Consistent behavior and easier testing |
| Agentic behavior | Constrain autonomy; no sensitive actions in MVP | Reduces risk and avoids unnecessary complexity |
| Integration | Framework-neutral widget; script tag is a future target | Keeps host integration simple without promising an unbuilt SDK |
| Context | Host supplies explicit, scoped context | Avoids guessing what product or feature the user means |
| Source policy | Cite evidence; clarify or abstain when insufficient | Builds trust and makes answer quality testable |
| Data retention | Do not store raw audio by default | Data minimization and lower privacy risk |
| Architecture | Modular monolith first | Clean boundaries without premature microservice operations |
| Testing | Golden evaluation set plus tenant/security tests | Measures answer quality and protects data boundaries |
| Delivery strategy | Working vertical slice before advanced features | Reduces risk and produces a demo early |

---

## 15. Contributing and development principles

- Prefer small, testable increments over broad abstractions.
- Keep secrets out of source control; `.env.example` contains placeholders only.
- Add tests whenever answer behavior, retrieval, access scope, or widget state changes.
- Keep provider-specific code behind adapters, but do not over-generalize prematurely.
- Every answer should have a clear status and traceable evidence when applicable.
- Never use synthetic demo citations to imply that real retrieval occurred.
- Document limitations and failed evaluations as well as successful demos.
- Use synthetic data in examples and tests.

## License

The repository should include an explicit open-source license before public release. MIT is a reasonable default for a portfolio project, provided it fits the author's intended distribution and contribution model.

---

## 16. What this project demonstrates

If completed well, JourneyHelp can demonstrate practical skills in:

- Product thinking and MVP prioritization.
- TypeScript widget/SDK design.
- Python API development.
- RAG ingestion, retrieval, citations, and evaluation.
- LLM provider abstraction and output validation.
- Voice UX and browser speech APIs.
- Context-aware product experiences.
- Security boundaries and multi-tenant isolation.
- Test-driven iteration, observability, and deployment documentation.

**The priority is not to maximize the number of AI components. It is to prove one useful product experience end to end, measure its reliability, and show that the same architecture can be reused in another product journey.**
