## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/159

**Issue title:** structlog output is not captured by pytest caplog — log assertions fail suite-wide

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
Structlog is not configured to propgate into the stdlib logging system in tests. Each test fails when asserting on caplog, even though the code itself runs fine.
The test_batch_processor.py unit test is currently broken. A fix would include configuring the structlog so caplog-based assertions work and these tests pass.



**Branch name:** bug/159-structlog-output-not-captured-by-pytest-caplog

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [link to commit documenting the reproduced issue]

**Failing test:** `tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_empty_chunks_list_returns_empty`

**Steps to reproduce:**
1. From the repo root, run the batch processor unit tests:
   ```
   python -m pytest tests/unit/test_batch_processor.py -v
   ```
2. Observe the result: `1 failed, 10 passed`.

**Reproduction summary:**
`test_empty_chunks_list_returns_empty` fails with `AssertionError: assert ('Empty chunks list' in '' or False)` — `caplog.text` is empty and `caplog.records` is empty. The captured stdout shows the warning *was* emitted (`[warning  ] Empty chunks list provided to BatchEmbeddingProcessor`), but structlog uses its default configuration (`structlog.get_logger()` in `batch_processor.py:7`) which prints straight to stdout instead of propagating through Python's stdlib `logging`. Since pytest's `caplog` fixture only captures stdlib `logging` output, the log record never reaches `caplog`, so the assertion fails even though the code runs correctly.