# Demo: offline run summary and CSV export

This recording uses **synthetic** run directories built from `ScoredAnswer` / `FailedAnswer` records plus a `config.json` shaped like a finished run. It does not use real benchmark rows.

The video is a replay of the exact captured command output in `commands.log`. The on-screen label is `Replay of real captured command output`.

## What the recording shows

1. Clean synthetic run: suite, search engine, model id/provider, counts, score, `TimeoutError` failure kind, and `saved summary: agrees`.
2. CSV export, `cat` of `task_id,status,correct,score,failure_kind`, and `ls -l` showing `-rw-------`.
3. Overwrite refusal (`exit: 1`) unless `overwrite=true`.
4. Malformed synthetic run: `task-c.json` reported and excluded from the score (`records found: 2`, score `0.75`).
5. `rg` of the planted sentinel `SYNTHETIC_SENTINEL_DO_NOT_EXPORT`: no match in stdout or CSV (`exit: 1`); matches only in private task files.

## Commands

The captured invocations are in `commands.log`. They were run as:

```sh
uv run --frozen python -m parallel_search_evals.summarize run_dir=/tmp/pse-demo/clean-synthetic
uv run --frozen python -m parallel_search_evals.summarize run_dir=/tmp/pse-demo/clean-synthetic csv=/tmp/pse-demo/clean-synthetic.csv
cat /tmp/pse-demo/clean-synthetic.csv
ls -l /tmp/pse-demo/clean-synthetic.csv
uv run --frozen python -m parallel_search_evals.summarize run_dir=/tmp/pse-demo/malformed-synthetic
```
