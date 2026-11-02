# ci-panel-perf-small-gha

**Frozen** test data for the CI Panel performance tests (MATLAB, UPT). The
tests only *read* this repo. It's the small baseline: loading it measures
the CI Panel's fixed per-request cost, which can be compared with
[`ci-panel-perf-large-gha`](https://github.com/mw-ci-test/ci-panel-perf-large-gha)
to separate data-volume cost.

**Do not push changes, add triggers, dispatch or re-run workflows** without
coordinating with the CI Panel QE owners. Any new run changes the data the
performance trend is based on.

## Shape

| | Count |
|---|---|
| Workflows | 1 (`perf-01`) |
| Runs | 3 |
| Jobs per run | 3 (matrix `shard-1` … `shard-3`) |
| Steps per job | 4: Set up job, Run shard, Upload artifact, Complete job (GitHub adds the first and last) |
| Artifacts per job | 1 (`shard-<n>`), `retention-days: 1`; the panel shows a run's 3 artifacts on its Run row |

As the CI Panel loads it: 1 + 3 runs + 9 jobs + 36 steps ≈ **49 tree
nodes**, from 1 + 1 + 2·3 = **8 REST requests** per load.

All jobs pass.

## Why it's built this way

Same as the large repo: `workflow_dispatch` only (nobody can add runs by
pushing or opening a PR); public repo on `ubuntu-latest` (free; don't make
it private or use larger runners); 1-day artifacts, so the repo freezes the
day after setup.

## One-time setup

Prerequisites: [GitHub CLI](https://cli.github.com/) logged in to github.com
as `mw-ci-test` (`gh auth login --hostname github.com`), and `jq`. On
Windows, run the scripts with Git Bash
(`& "C:\Program Files\Git\bin\bash.exe" scripts/dispatch-runs.sh`), not WSL.

1. Push this content to `main`.
2. Create the runs (3 dispatches; safe to re-run, since it only adds missing
   runs):
   ```bash
   scripts/dispatch-runs.sh
   ```
3. Wait for the 3 runs to finish (a few minutes).
4. Check the shape (read-only):
   ```bash
   scripts/verify.sh
   ```
   Expected: 1 workflow, 3 completed runs, 3 jobs per run, 4 steps per job.
   Artifacts read 3 per run until they expire, then 0.

If a run fails for an unexpected reason, **don't re-run it**: delete it in
the web UI, run `scripts/dispatch-runs.sh` again, and check with
`scripts/verify.sh`.

## Token for the tests

The perf tests read this repo with a fine-grained PAT owned by
`mw-ci-test`, limited to the perf repos, with **Actions: Read** and
**Metadata: Read** permissions. It's stored in TCWS as
`CIPANEL_PERF_GHA_TOKEN`.
