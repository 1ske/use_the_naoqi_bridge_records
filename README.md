# use_the_naoqi_bridge_records

Run records of the evaluation in the master's thesis *Task-Level Control of a Humanoid Robot using the Model
Context Protocol: A Comparison of LLMs and Agent Harnesses* (Isak Greatrex, Linköping University, 2026). The system and the scripts that analyse
these records are in [use_the_naoqi_bridge](https://github.com/1ske/use_the_naoqi_bridge); the naoqisim setup for
the simulated NAO is in [NAOQI_ATTEMPT_v1](https://github.com/1ske/NAOQI_ATTEMPT_v1).

Eight agents (four LLMs, each in its native harness and in OpenCode) did three tasks: Corridor and Complex
corridor on the simulated NAO (Webots + naoqisim) and Search on the physical NAO V5.

```
grid/
  main/runs/<task>-<agent>-<UTC time>/   257 main runs: 240 valid, 17 invalid (repeated in place)
  main/orders/<task>.json                the randomized run order (blocks, seed)
  main/analysis/, main/figures/          the tables and figures of thesis Chapter 5, as generated
  main/logs/                             runner logs
  pilot/runs/, pilot/orders/, ...        the 60 pilot runs of 2026-10-15 (thesis section 4.12; not analysed)
  checks/<agent>-<check>-<when>-<time>/  instruction and image checks before the pilot and after the main runs
```

Per run: `manifest.json` (agent, harness and version, system commit, world file and hash, prompt, caps, end
condition), `calls.jsonl` (the bridge's per-call log), `agent/` (the harness's transcript; for Antigravity its
`agy-brain/` conversation logs), `transcript_check.json`, and `ground_truth.json` + `trajectory.jsonl`
(simulated tasks), `operator_notes.json` (Search: the operator's verdict, falls and interventions) or `cost.json`
(OpenCode: OpenRouter usage). Valid/invalid and success are computed by `pilot/metrics.py`, not stored.

## What was removed before publication

- **Camera images of the physical NAO (Search runs, main and pilot).** They show the shared lab, the people
  working in it and their screens. Removed: the image files (`operator_images/`, Antigravity's saved images) and
  the images embedded in transcripts, which now read `<camera image removed: physical NAO in a shared lab>`. All
  text, including the LLMs' descriptions of what they saw, is kept. Images of the simulated runs and the synthetic
  code images of the image checks are kept.
- **The stand-in LLM's request logs** (`checks/*-standin-*/.../fake_llm_requests.jsonl`) contained the harnesses'
  full requests, including the vendors' system prompts. Each is replaced by `fake_llm_requests.summary.jsonl`:
  per request, the API, the user agent, whether the server's instructions and an image were in it, and the usage.
- The robot's LAN address is replaced by `<nao-lan-ip>` and local home paths by `~`. Credentials were never
  recorded in clear (the harnesses' authorization headers were redacted when recorded).
- The operator's notes in `operator_notes.json` were written in a chat during the runs. Where they quoted that
  chat in informal wording, they are rewritten in plain wording; the distances, facing judgements, falls and
  verdicts are unchanged.
- Runs made only to develop the instrumentation (phase `test`) and a void earlier pilot with other agents are
  not included.

The analysis does not use any of the removed parts: running the scripts below on these records gives the same
tables as in the thesis, byte for byte (checked 2026-11-02).

## Reproducing the tables

The scripts read `~/.naoqi-pilot/grid`, so point that at this folder:

```
mkdir -p ~/.naoqi-pilot && ln -s "$PWD/grid" ~/.naoqi-pilot/grid
cd ../use_the_naoqi_bridge
uv run --script pilot/analysis.py --phase main --out /tmp/analysis   # Chapter 5 tables
uv run --script pilot/posthoc.py --phase main --out /tmp/analysis    # section 5.6 tables
uv run --script pilot/figures.py --phase main --out /tmp/figures      # Chapter 5 figures
```

The protocol was fixed with the tag `protocol-main-2026-10-19` in use_the_naoqi_bridge; each manifest records the
system commit of its run.

## License

CC BY 4.0 (see `LICENSE`). The transcripts include text written by the LLMs and tool descriptions from the
harnesses as they appeared in the runs.
