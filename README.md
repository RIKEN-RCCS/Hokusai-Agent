# HOKUSAI BigWaterfall2 (HBW2) agent plugin

An MCP plugin (Claude Code / Codex) for **HOKUSAI BigWaterfall2 (HBW2)**, the
RIKEN R-CCS CPU-first Slurm cluster. It lets an agent submit and monitor batch
jobs, manage files, and search a hand-written machine guide — describing work
in resource terms (nodes / processes / threads / memory / wall time / project)
rather than writing sbatch scripts by hand.

This repo is a thin "skin" over
[`hpc-agent-core`](https://github.com/william-dawson/hpc-agent-core), which
provides the generic runtime (SSH middleware, PSI/J-style job models, the
Slurm backend, the docs RAG pipeline, health checks, and MCP serving glue).

## What HBW2 is

- **MPC** — 312 nodes × 112 Intel Xeon cores, ~112 GiB each; the workhorse.
- **LMC** — 2 nodes, ~2.7 TiB each; single-host large-memory jobs.
- **GPU** — 4 nodes, NVIDIA H100; secondary, mainly postprocessing.

Slurm-scheduled, reached over SSH (key-only) via `hokusai1`–`hokusai4`. Every
job is billed to a project under a fair-share policy. See
[`docs/hokusai_guide.md`](docs/hokusai_guide.md) for the full orientation.

## Install

The MCP servers live in [`server/`](server/) as the `hokusai-mcp` package.

```bash
cd server
python3 -m venv .venv && .venv/bin/pip install -e .
# or, to expose the servers on your PATH for the plugin: pipx install ./server
```

This provides three console scripts: `hokusai-hpc` (the job/filesystem MCP
server), `hokusai-docs` (the docs-search MCP server), and `hokusai-doctor`
(health checks). The plugin's [`.mcp.json`](plugins/hokusai/.mcp.json)
launches the first two.

## Configure

The plugin reads `~/.hpc-agent/hokusai.json`:

```json
{
  "ssh": { "host": "hokusai" },
  "defaults": { "account": "RB99999" }
}
```

- `ssh.host` — an alias in `~/.ssh/config` or `user@hokusai.riken.jp`
  (register your SSH key via the portal `https://hokusai.riken.jp/hbw2/`
  first; auth is key-only).
- `defaults.account` — the project charged when a job names none;
  **mandatory** on HBW2. `RB…` (RIKEN) or `HP…` (HPCI).

Env overrides: `HOKUSAI_HOST`, `HOKUSAI_ACCOUNT`, `HOKUSAI_CONFIG`. The
`hokusai-configuring` skill walks a user through this.

## Verify

```bash
cd server
.venv/bin/hokusai-doctor            # config, SSH+Slurm, guide, docs index, embedding
.venv/bin/python tests/smoke.py     # read-only MCP stdio test
.venv/bin/python tests/smoke.py --job   # + submits a real tiny job (needs a project)
```

The embedding check is optional — off the RIKEN network the docs server falls
back to keyword (BM25) search.

## Layout

- `server/hokusai_mcp/` — the plugin package (config, compute, MCP servers,
  bundled `data/`).
- `plugins/hokusai/` — Claude Code / Codex plugin manifests, `.mcp.json`, and
  skills.
- `docs/hokusai_guide.md` — the machine guide (also bundled under the package
  as `data/hokusai_guide.md`, which is what the docs index is built from).
- [`AGENTS.md`](AGENTS.md) — design rules, cluster facts, repo map.
- [`IRI_CHECKLIST.md`](IRI_CHECKLIST.md) — IRI Facility API coverage.

See [hpc-agent-core's `PORTING.md`](https://github.com/william-dawson/hpc-agent-core/blob/main/PORTING.md)
for the general porting process this repo follows.
