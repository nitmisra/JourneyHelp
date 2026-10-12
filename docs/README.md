# JourneyHelp

**An embeddable, context-aware product assistant.** JourneyHelp helps users understand and use a product without leaving their current journey.

This repository contains the first end-to-end MVP slice: a simulated automotive dashboard, reusable TypeScript Web Component, FastAPI endpoint, lightweight knowledge retrieval, optional OpenAI integration, and tests.

> **Safety note:** This is a portfolio prototype using illustrative sample content. It is not a vehicle diagnostic tool and does not connect to or control a real vehicle.

## What this MVP does

- Lets users select one of four simulated dashboard items.
- Passes the selected item and product context to JourneyHelp.
- Accepts a text question through an embeddable Web Component.
- Retrieves relevant entries from a sample-manual knowledge base.
- Returns an answer and source references.
- Uses OpenAI when configured; otherwise uses a clearly labelled local responder.
- Includes tests that do not require an API key.

## Architecture

```text
Automotive demo (Vite + TypeScript)
  ├─ simulated dashboard and selected item
  └─ <journey-help-widget>
           │ POST /v1/chat
           ▼
       FastAPI backend
           ├─ request validation
           ├─ keyword retrieval
           ├─ OpenAI adapter (optional)
           └─ local deterministic responder
           ▼
       answer + status + source references
```

## Quick start

Requirements: Python 3.10+, Node.js 20+. An OpenAI API key is optional.

### 1. Start the API

From the repository root:

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\\Scripts\\activate
pip install -r services/api/requirements.txt
cp .env.example .env
uvicorn services.api.app.main:app --reload --port 8000
```

Check `http://localhost:8000/health`.

### 2. Start the automotive demo

In a second terminal:

```bash
cd apps/demo
npm install
npm run dev
```

Open the Vite URL shown in the terminal, normally `http://127.0.0.1:5173`.

### 3. Try these questions

Select a dashboard item and ask:
- What does this warning mean?
- How do I use the rear defogger?
- What does the engine-shaped indicator mean?
- Can I keep driving if the oil pressure warning is red?

Critical questions should receive cautious guidance. The demo does not diagnose a real vehicle.

## Configure OpenAI

Set these values in `.env` and restart the API:

```dotenv
OPENAI_API_KEY=your_key_here
OPENAI_MODEL=gpt-4o-mini
```

The key is read by the backend. Never put it in browser code or commit it. Provider availability and pricing depend on your account.

When no key is configured, the API uses `local-demo-responder`. This makes the demo testable without external API calls.

## Run tests

From the repository root:

```bash
pip install -r services/api/requirements.txt
pytest -q
```

## Repository map

```text
JourneyHelp-SDD/
├── README.md                  # project overview and setup
├── PRD.md                     # product requirements and acceptance criteria
├── specs/                     # feature-level SDD specifications
├── apps/demo/                 # Vite automotive demo
├── packages/widget/           # TypeScript Web Component
├── services/api/app/          # FastAPI and answer engine
├── examples/automotive/       # illustrative knowledge entries
├── tests/                     # automated tests
└── docs/                      # architecture, decisions, security, evaluation
```

## Functional scope

**Included:** dashboard selection; context propagation; `POST /v1/chat`; `/health`; keyword retrieval; optional OpenAI integration; local fallback; source references; cautious safety responses; tests.

**Not included:** voice I/O; pgvector or document ingestion; authentication, tenant management, admin console, analytics, durable history, production deployment, or real vehicle data.

## API request example

```json
{
  "project_id": "automotive-demo",
  "session_id": "demo-session",
  "message": {"type": "text", "content": "What does this warning mean?"},
  "context": {"product":"Journey Motors Atlas", "model_year":"2025", "page":"/dashboard", "selected_item":"tire-pressure", "locale":"en-IN"},
  "response_preferences": {"mode":"text"}
}
```

Response includes `status`, `answer`, `sources`, `response_mode`, `needs_mode_selection`, `demo_mode`, and `model`.

## Specification-Driven Development

Treat `PRD.md` as the product-level source of truth and feature files under `specs/` as implementation contracts. For every change:

1. Review the feature spec.
2. Confirm scenarios and acceptance criteria.
3. Update API/data contracts if needed.
4. Write tests from the criteria.
5. Implement the smallest conforming change.
6. Run tests and validate the demo.
7. Update specs and decision records when behavior changes.

## Key decisions and rationale

| Decision | Why |
|---|---|
| Automotive first | Concrete, memorable context-aware demo |
| Simulated dashboard | Avoids real-vehicle dependencies and safety risks |
| TypeScript Web Component | Framework-independent embedding boundary |
| FastAPI | Typed API and quick Python iteration |
| One optional OpenAI provider | Proves an LLM path without premature abstraction |
| Keyword retrieval first | Establishes a baseline before vector-search complexity |
| Local responder | Reproducible demo without credentials |
| Backend-owned source references | Prevents model-invented citation IDs |
| Text first, voice later | Proves the core loop before speech complexity |
| Modular monolith | Easier to build and debug |

## Security and privacy

Never commit `.env` or credentials. Treat user input and retrieved documents as untrusted. The local demo is not production-hardened. Before public hosting, add authentication, authorization, rate limits, monitoring, data retention controls, and stronger origin/security controls. Use synthetic data only.

## Roadmap

1. Validate the text-based end-to-end slice.
2. Improve retrieval evaluation and test coverage.
3. Add document ingestion and versioning.
4. Add voice input/output using the same answer engine.
5. Add tenant isolation, authentication, analytics, and production hardening.
6. Demonstrate reuse with banking and SaaS journeys.

## License

MIT. See `LICENSE`.
