# SPEC-005: Optional OpenAI integration

Read `OPENAI_API_KEY` and `OPENAI_MODEL` on the server. Send question, product context, and retrieved evidence. Instruct the model to ground factual claims and abstain when evidence is missing. Never expose the provider key to the browser. Return a controlled 502 on provider failure. Default tests must not make real provider calls. Without a key, use the clearly labelled deterministic local responder.
