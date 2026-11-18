Title: Add an offline run summary command with CSV export

## What changed

A finished run directory could only be summarized by invoking `run_eval`, which requires credentials and refuses to start without them. Copied directories and interrupted runs could not be re-read offline.

`python -m parallel_search_evals.summarize run_dir=...` now reprints suite, search engine, model id/provider, `source_sha256`, readable-record counts, score (failures count as zero, via `EvaluationSummary.from_records`), failure kinds, and malformed task files that would rerun on resume (excluded from the score). If `results/{run_name}.json` exists beside the run, the command reports whether it agrees with the recomputed numbers. Optional `csv=` writes `task_id,status,correct,score,failure_kind` sorted by task id, using the existing 0600 atomic writer, and refuses to overwrite unless `overwrite=true`. Question text, references, answers, grade text and native history are never printed or exported.

The attached recording uses synthetic run directories, not real benchmark rows.

## Validation

Untouched upstream tip `6d29a42`:

- `uv run --frozen ruff check src tests .github/quality/check.py .github/scripts` — all checks passed
- `uv run --frozen ruff format --check src tests .github/quality/check.py .github/scripts` — 103 files already formatted
- `uv run --frozen mypy --strict --no-incremental src/parallel_search_evals` — Success: no issues found in 27 source files
- `uv run --frozen pytest -q` — 1385 passed, 2 failed (`test_https_and_literal_loopback_actual_sdk_dispatch_preserves_endpoint` for `http://[::ffff:127.0.0.1]/v1` responses and chat). Those two failures were present on untouched upstream.
- `uv run --frozen python .github/scripts/check_docs.py` — `{"local_links": 29, "parsed_commands": 4, "failures": []}`
- `uv run --frozen python .github/quality/check.py --output metrics.json` — gates_pass true; mean CC 2.964, mean MI 70.028, isolated Ruff 6, 27 modules, 112 functions
- `uv run --frozen python .github/scripts/check_public.py` — 0 findings
- `uv build` — wheel and sdist built
- `uv run --frozen python .github/scripts/check_package.py` — passed
- `uvx --from pre-commit==4.7.0 pre-commit run --config .github/pre-commit-config.yml --hook-stage manual --all-files` — gitleaks checkout and history passed

After this change:

- ruff check — all checks passed
- ruff format — 105 files already formatted
- mypy — Success: no issues found in 28 source files
- pytest — 1392 passed, 2 failed (the same two IPv4-mapped loopback cases as upstream; 7 new tests in `tests/test_summarize.py` passed)
- check_docs — `{"local_links": 29, "parsed_commands": 4, "failures": []}`
- quality — gates_pass true; mean CC 2.855, mean MI 71.099, isolated Ruff 6, 28 modules, 131 functions
- check_public — 0 findings
- uv build / check_package — passed
- gitleaks checkout and history — passed

Not verified: paid provider runs, real DSQA/HLE rows, and whether `ip_address("::ffff:127.0.0.1").is_loopback` is true on this host (the two failing tests predate this change).

## Evaluation impact

None. The command only reads finished run directories, existing runs resume unchanged and results stay comparable.

## Demo

![demo](https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1117/run-summary-export/preview.gif)

Full recording (MP4): https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1117/run-summary-export/out.mp4
