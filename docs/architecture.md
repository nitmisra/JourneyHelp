# JourneyHelp Architecture

## Runtime flow

1. A host page embeds the widget and supplies approved product/page context.
2. The user types a question or uses push-to-talk.
3. Speech is transcribed by a speech adapter; text input skips this step.
4. The API validates request shape, project scope, origin policy, and rate limits.
5. The answer engine retrieves relevant chunks filtered to the authorized project and product context.
6. An evidence check determines whether to answer, clarify, or abstain.
7. The LLM composes an answer from retrieved evidence.
8. Output validation checks status, source references, and applicable safety rules.
9. The widget displays text or speaks the answer, showing a transcript for spoken output.

## Design boundaries

- The widget handles interaction state, not secrets or access control.
- The API is the security boundary.
- The answer engine does not independently authorize data access.
- Ingestion is responsible for parsing, chunking, metadata, indexing, and versioning.
- Speech is replaceable and must have a text fallback.
- Analytics should collect only what is necessary to improve the experience.

## Initial deployment shape

Use a modular monolith for local development: one API process and PostgreSQL/pgvector. The widget and demo UI are built as front-end packages. Split services only when operational or scaling evidence justifies the added complexity.
