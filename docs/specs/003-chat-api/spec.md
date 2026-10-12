# SPEC-003: Chat API

## Endpoints
- `GET /health`
- `POST /v1/chat`

## Requirements
Validate typed request; retrieve evidence; generate response; return status, answer, sources, response mode, demo flag, and model label. Provider errors must not leak secrets or stack traces.

## Errors
- 422: request validation failure
- 502: configured provider failed
- 500: unexpected server failure
