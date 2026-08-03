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

**Failing test:** `tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_empty_chunks_list_returns_empty`

**Steps to reproduce:**
1. From the repo root, run the batch processor unit tests:
   ```
   python -m pytest tests/unit/test_batch_processor.py -v
   ```
2. Observe the result: `1 failed, 10 passed`.

**Reproduction summary:**
`test_empty_chunks_list_returns_empty` fails with `AssertionError: assert ('Empty chunks list' in '' or False)` — `caplog.text` is empty and `caplog.records` is empty. The captured stdout shows the warning *was* emitted (`[warning  ] Empty chunks list provided to BatchEmbeddingProcessor`), but structlog uses its default configuration (`structlog.get_logger()` in `batch_processor.py:7`) which prints straight to stdout instead of propagating through Python's stdlib `logging`. Since pytest's `caplog` fixture only captures stdlib `logging` output, the log record never reaches `caplog`, so the assertion fails even though the code runs correctly.

**PLAN.md link:** https://github.com/Nexus-00/pathreview/blob/bug/159-structlog-output-not-captured-by-pytest-caplog/PLAN.md

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
1. The autouse configure_structlog_for_tests fixture in tests/conftest.py configures structlog with stdlib.LoggerFactory(), cache_logger_on_first_use=False, and a plain ConsoleRenderer(colors=False).

2. The fixture sets the root logger to DEBUG so WARNING-level records propagate to caplog's handler. I verified via the target test, which asserts on both caplog.text and caplog.records[*].message — both are populated, and it passes.

**Next steps:**
Implementing the rest of the plan, through steps 3 to 5: Run the target test, run the full suite, commit, then submit the PR.

**Blockers:**
Time management is an issue. I've been busy throughout the entire week, and did not make as much progress as I wanted to in the middle of the week.

---

### Check-in 2 (end of week)

**PR link:** [link to your submitted pull request]

**Branch:** bug/159-structlog-output-not-captured-by-pytest-caplog

**What you built:**
Added an autouse fixture in `tests/conftest.py` that reconfigures structlog to route events through Python's stdlib `logging` (via `structlog.stdlib.LoggerFactory()` with `cache_logger_on_first_use=False` and a plain, non-ANSI `ConsoleRenderer(colors=False)`) and raises the root logger to `DEBUG` so WARNING records reach pytest's caplog handler. Because structlog config is process-global, every module using `structlog.get_logger()` now has its logs captured by caplog during tests, and the fixture calls `structlog.reset_defaults()` on teardown to avoid config bleed.

**Tests added or updated:**
`tests/conftest.py` — added the `configure_structlog_for_tests` autouse fixture; no test assertions were changed. This unblocks the existing failing test `tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_empty_chunks_list_returns_empty` (which asserts on both `caplog.text` and `caplog.records`) and enables caplog-based assertions suite-wide.

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes
- `make test-unit`: the fix takes the unit suite from 53 failing to 52 (fixes the target test, zero new failures). The remaining 52 failures are pre-existing and unrelated to #159 (async-mock setup and domain-logic issues in `review_service`, `resume_parser`, `tech_detector`, etc.).
- `make check`: `tests/conftest.py` is clean under `ruff` and `black`; `mypy`'s `typecheck` target does not cover `tests/`. The 182 `make check` errors are all pre-existing and outside the scope of this issue.

**Draft PR feedback received from:** none
