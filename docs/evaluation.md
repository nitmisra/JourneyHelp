# Evaluation Plan

## Golden-set categories

Create 50–100 synthetic questions across automotive and SaaS.

1. Direct answer in one chunk.
2. Answer requiring multiple chunks.
3. Ambiguous product, model, year, or symbol.
4. Unsupported question.
5. Conflicting or outdated source.
6. Similar models with different instructions.
7. Prompt-injection text embedded in a document.
8. Cross-project access attempt.
9. Voice transcription typo or noisy transcript.
10. Mode selection, stop, replay, and transcript behavior.

## Metrics

- Retrieval relevance / hit rate
- Grounded-answer rate
- Correct abstention and clarification rate
- Citation validity
- Context-use accuracy
- Tenant-isolation pass rate
- Voice task completion
- Latency and cost per answer

## Reporting rules

- Do not claim a percentage until the test set has been run.
- Keep expected source IDs and answerability labels with each case.
- Review failures by category rather than relying on one aggregate score.
- Add a regression test for each serious failure found.
