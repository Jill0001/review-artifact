# Agent as Policy for Robotic Manipulation

Anonymous code accompanying the submitted paper.

## What is here

| Path | Contents |
|---|---|
| `agp/` | Experiment harness: session server (`server_real.py`), the agent's CLI (`robot_client.py`), prompts and interface contracts, the runners, and the bridge launcher `start_bridges.sh` |
| `agp/tools/` | Session helpers and the metrics scripts that produce the paper tables |
| `agp/knowledge/`, `agp/paper_runs/` | The note/tool stores carried between trials, and the per-batch `results.csv` with before/after photos |
| `hardware-bridge/` | Safety-owned bridge that exclusively owns the arms and cameras — its own uv project, documented in `hardware-bridge/README.md` |
| `calib/` | Hand-eye and overhead-camera calibration procedures (`calib/README_runbook_zh.md`) |
| `third_party/graph-as-policy/` | The vendored connector package (`gap`, `gap_core`), Apache-2.0, pinned to one upstream commit (see its `UPSTREAM.md`) |
| `third_party/i2rt/` | Patch + pinned upstream commit for the robot SDK (see its `UPSTREAM.md`) |

The bridge ships as the Python package `agp_yam_bridge` with three console
scripts: `agp-yam-preflight`, `agp-yam-bridge`, `agp-yam-camera-acceptance`.

## Getting the robot SDK

The bridge depends on a patched i2rt checkout at `<repo>/i2rt` (its uv source is
`../i2rt`). Clone it there, pin the commit, apply our patch **without committing
it**, then copy the untracked additions; the exact commands are in
`third_party/i2rt/UPSTREAM.md`.

## Running

```bash
# 0. once per checkout, after the SDK step above: build the bridge environment. It is
#    also the interpreter the harness runs on, and it is what makes `gap` importable.
(cd hardware-bridge && uv sync --locked)

# 1. bring up a bridge (per arm; --source fake needs no hardware — see hardware-bridge/README.md)
bash agp/start_bridges.sh start left
bash agp/start_bridges.sh status

# 2. a goal set for the task: a photo of the goal state, or a recorded human demonstration
G=agp/goal_sets/$(date +%Y%m%d)_<task>; mkdir -p $G
cp <photo>.jpg $G/test_1.jpg && bash agp/capture_top.sh $G/top_camera.png
bash agp/record_demo.sh <task>

# 3. one supervised trial: it takes the before-photo, then waits for Enter before the arm moves
bash agp/run_trial.sh <task> <trial_no> [--effort high] [--model <slug>] \
     [--bare] [--right | --dual] [--knowledge task|ckpt|mx]
PAPER_DRYRUN=1 bash agp/run_trial.sh <task> 1   # print the resolved config, start nothing

# 4. results in agp/paper_runs/<task>_<batch>/ (results.csv, before/after photos);
#    the session (videos, joints, agent events) in agp/sessions/
bash agp/start_bridges.sh stop left
```

Large artefacts referenced by these steps (goal sets, demo videos, session
recordings) are not included in this code archive; use locally collected inputs. No anonymous data download is configured.

## License

Code in this repository is Apache-2.0 (`LICENSE`); `NOTICE` carries the
third-party attributions that come with it.  The vendored connector under `third_party/graph-as-policy/` is
Apache-2.0 and modified, upstream `LICENSE` and `NOTICE.md` preserved. The
vendored SDK under `third_party/i2rt/` remains MIT, upstream notice preserved.


## Anonymous review copy

This archive contains source code and the existing release tables. Author identifiers and links to identifying project resources are omitted. Device serial numbers are placeholders. Hardware configurations and calibration examples must be adapted and recalibrated for the reader's own setup before physical use. The software safety and preflight checks remain enabled.

The SDK patch contains anonymized comments; its patch fingerprint, runtime configuration references, and file digests have been regenerated. Large input data and recordings are not bundled.

Reproduction limitation: the original fixed i2rt commit documented in `third_party/i2rt/UPSTREAM.md` is currently unavailable from its public upstream. No replacement dependency version has been selected. See that file for the validation scope. Historic calibration-only configurations retain their original, separate fingerprints; they are not validated with the runtime patch.
