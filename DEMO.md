# Paired run compare demo

This recording is a replay of real captured command output from
`uv run --frozen python -m parallel_search_evals.compare` against synthetic
run directories. The directories were built from the harness record types
(`ScoredAnswer` / `FailedAnswer` plus a `config.json` shaped like
`run.run_evaluation`). They contain no live benchmark rows.

The banner on every frame reads **Replay of real captured command output**.
Every command, line of output, and exit code in the video matches `commands.log`.

## What the recording shows

1. A matched synthetic baseline vs search pair with `ids=true`: each side's
   score, the paired table, failures counted as incorrect, net change, the
   exact two-sided McNemar (binomial) p-value, and discordant task IDs.
2. The same pair with `json=true` piped to `python3 -m json.tool`.
3. A refused comparison against a run whose `source_sha256` differs
   (`exit_code=1`).
4. A refused comparison whose candidate is missing two task records
   (`exit_code=1`, missing IDs named).

Media: `out.mp4` is 1920x1080, 30 fps, H.264, yuv420p, 24 s.
`preview.gif` is 1280x720, 10 fps, 339074 bytes. Probe output is in
`logs/ffprobe.txt`.
