Title: Compare two run directories question by question

## What changed

Baseline and search are still invoked as separate runs, but the harness only
printed each directory's own aggregate score. There was no way to see which
questions search fixed or broke, or to check that the two directories really
covered the same questions with the same source and judge.

`python -m parallel_search_evals.compare` reads two finished run directories
and prints a question-by-question report. It needs no credentials and opens no
client. Before any scores, it requires matching `suite`, `source_sha256`,
`source`, `synthetic`, `judge_model`, `judge_assets` (when present),
`filter_benchmarks` and `benchmark_sources`. It prints which of `search_engine`
and model settings differ. Both directories must contain the same readable
task IDs; otherwise it names the IDs missing on each side (count and first
few) and exits non-zero.

On a completed comparison it reports each side's score/correct/failed via
`EvaluationSummary.from_records`, the paired table (both correct, only base
correct, only candidate correct, neither), failures counted as incorrect and
shown separately, the net change, and an exact two-sided McNemar (binomial)
p-value on the discordant pairs computed with `math.comb`. `json=true` prints
a machine-readable object. `ids=true` lists the task IDs in each discordant
cell. Output never includes question text, answers, references, grade text or
history.

Missing directories, unreadable configs, unmatched identity fields, and
unmatched task sets each produce one clear message and a non-zero exit.
A completed comparison exits 0.

## Validation

Recorded first on untouched `upstream/main` (`6d29a42263c24be96211e757940c604fca941972`), then again on this change.

- `uv run --frozen ruff check src tests .github/quality/check.py .github/scripts` — All checks passed, before and after.
- `uv run --frozen ruff format --check src tests .github/quality/check.py .github/scripts` — 103 files already formatted before; 105 after.
- `uv run --frozen mypy --strict --no-incremental src/parallel_search_evals` — Success: no issues found in 27 source files before; 28 after.
- `uv run --frozen pytest -q` — before: 1385 passed, 2 failed; after: 1413 passed, 2 failed. `tests/test_compare.py`: 28 passed. The two failures are `tests/models/test_endpoint_security.py` IPv4-mapped loopback cases (`http://[::ffff:127.0.0.1]/v1` responses and chat). They fail on untouched upstream under this Python 3.12.3 (`ipaddress` does not treat that mapped address as loopback) and are unchanged by this work.
- `uv run --frozen python .github/scripts/check_docs.py` — `{"local_links": 29, "parsed_commands": 4, "failures": []}` before and after.
- `uv run --frozen python .github/quality/check.py --output metrics.json` — `gates_pass: true` before and after.
  - Before: mean_complexity 2.9642857142857144, mean_mi 70.02824520564647, ruff_count 6, module_count 27, function_count 112.
  - After: mean_complexity 2.968503937007874, mean_mi 71.09866501973053, ruff_count 6, module_count 28, function_count 127.
- `uv run --frozen python .github/scripts/check_public.py` — findings `[]` before and after.
- `uv build` — wheel and sdist built.
- `uv run --frozen python .github/scripts/check_package.py` — passed (compare.py is in the wheel).
- `uvx --from pre-commit==4.7.0 pre-commit run --config .github/pre-commit-config.yml --hook-stage manual --all-files` — Scan checkout for secrets passed; Scan commit history for secrets passed.

Full command transcripts are in `logs/` on `demo/paired-run-compare`.

## Evaluation impact

None. The command only reads finished run directories; existing runs resume unchanged and results stay comparable.

## Demo

![demo](https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1117/paired-run-compare/preview.gif)

Full recording (MP4): https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1117/paired-run-compare/out.mp4
