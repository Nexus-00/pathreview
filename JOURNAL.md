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