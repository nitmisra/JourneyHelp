# SPEC-002: Embeddable chat widget

## Objective
Provide a framework-independent text interface.

## Requirements
- Register `<journey-help-widget>` as a Web Component.
- Display welcome message, user messages, answers, sources, loading state, and recoverable errors.
- Prevent duplicate submissions while a request is active.
- Render all content as text, never executable HTML.
- Provide accessible labels and responsive layout.

## Scenarios
- Valid question: show answer and source references.
- Pending request: prevent duplicate submit.
- API error: show recoverable error and re-enable input.
