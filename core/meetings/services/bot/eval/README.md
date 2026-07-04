# @vexa/bot — eval (the bot-local L4 harness)

Validate + stress-test the **standalone carved bot** against a **live Google Meet**, with synthetic
speaker-bots, a **live eyeball viewer**, and an **autonomous PASS/FAIL verdict**. The bot is treated as
a **module whose only difference is it's a runnable service**: one command in, one verdict out.

It reuses the shared `meetings/eval` machinery in place (speakers · launch · drive · corpus · noise ·
analyze · judge) — nothing is forked — and adds only the bot-targeted runner, the viewer, and the
verdict/attribution glue.

## One command

```bash
make -C meetings/services/bot/eval run MEETING=rvf-kywf-pxb
```

What it does: starts the **viewer** (`http://localhost:8090`) → spawns the bot on the configured bot
host (`BBB_HOST`, see `config.env.example`) into the Meet → bridges its `lifecycle.v1` +
`transcript.v1` live to the viewer → (you admit the bot once) → drives synthetic speakers → scores the
bot's transcript against the eval baseline → prints `VERDICT PASS|FAIL` (and the suspected upstream
brick on red).

## Pieces


| File                                      | Role                                                                                                                                                                     |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `run.sh`                                  | the orchestrator (viewer + remote bot + feed bridge → drive → score → verdict). `make run MEETING=<id>`.                                                                 |
| `viewer/server.mjs` · `viewer/index.html` | the live eyeball — a dumb SSE sink (POST `/lifecycle` `/transcript` `/verdict`) + a 1-page UI: transcript feed · lifecycle timeline · verdict banner. No deps, no build. |
| `feed.mjs`                                | bot-host→viewer bridge: `docker logs -f` → `/lifecycle`; poll the `transcription_segments` redis stream → `/transcript`.                                                 |
| `verdict.mjs`                             | the autonomous oracle: runs `analyze.mjs` (SCORE) + `judge.py` (JUDGE) against the eval baseline → one PASS/FAIL, POSTs the banner, exits 0/1.                           |
| `attribute.mjs`                           | on red, maps the failing metric → the upstream `@vexa/*` brick + the offline `gate:replay` repro command.                                                                |
| `verify.sh`                               | offline self-test of the oracle (clean→PASS, misattr→FAIL→brick). `make verify`.                                                                                         |
| `config.env.example`                      | bot-host topology + secrets path + run knobs. `cp` → `config.env` (gitignored).                                                                                          |


## Verify offline (no meeting, no remote host)

```bash
make -C meetings/services/bot/eval verify     # gates: clean fixture → PASS, misattr fixture → FAIL → brick
PORT=8097 node viewer/server.mjs &            # then curl /lifecycle /transcript /verdict and open :8097
```

## The verdict (gmeet lane)

HARD: `misattr=0` · `dup=0` · `seg_N=0` (fully speaker-bound) · `leakage=0` · `hijack=0` (noise lane).
SOFT: oversegmentation `midcut/segments ≤ 10%`. `completeness`/`attribution_pct` are reported, not
hard-gated (attribution over-counts under `/speak` latency). `0 segments` ⇒ FAIL.

## Separating concerns (debug the right module)

A red verdict points at ONE brick (`attribute.mjs`): join → `@vexa/join`; silent → capture /
`@vexa/gmeet-capture`; no text → `@vexa/transcribe-whisper`; misattr/overseg → `@vexa/gmeet-pipeline`.
Reproduce it OFFLINE + deterministically through the real pipeline — no live Meet, no STT, no server —
with `pnpm --filter @vexa/bot run replay` (`gate:replay`, `../src/replay.test.ts`).

## Prereqs

SSH access to a Docker host running the compose stack (redis, runtime-api, shared docker network) with
the bot image present; a secrets env file providing `VEXA_BASE`, `PLATFORM`, `NATIVE_ID`, and per-speaker
API tokens. Everything is a knob — copy `config.env.example` to `config.env` (gitignored) and point it
at your rig. Never commit secrets.
