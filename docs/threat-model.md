# Initial Threat Model

This is a starting checklist, not a formal security assessment.

## Assets
- Project and tenant knowledge
- Provider credentials
- Admin configuration
- User questions and session metadata
- Ingestion sources and document versions

## Threats and mitigations

| Threat | Initial mitigation |
|---|---|
| Cross-project retrieval | Enforce tenant/project filters server-side on every retrieval and data query; add negative tests |
| Leaked provider key | Keep credentials server-side; use environment variables; never expose secrets in widget config |
| Prompt injection in documents | Treat retrieved text as untrusted data; never let it override system policy or grant tools |
| Malicious or oversized input | Validate schema and size; rate-limit; use request timeouts |
| Arbitrary URL ingestion / SSRF | Restrict ingestion sources and validate URLs; block private/internal network targets in any URL fetcher |
| Fabricated citations | Cite only retrieved source IDs; validate IDs and section references |
| Stale manual content | Store version/effective date/status; provide source disable and reindex path |
| Sensitive logs | Minimize content logging; avoid raw audio; define retention and deletion |
| Public project ID mistaken for a secret | Treat it as public; enforce authorization and scope on the server |
| Unsafe automotive guidance | Use approved vehicle documentation; abstain on uncertainty and escalate safety-critical cases |

## MVP privacy defaults
- Use synthetic data in the public demo.
- Do not store raw audio by default.
- Avoid private account or vehicle telemetry integrations.
- Make retention behavior explicit before adding persistent conversation history.
