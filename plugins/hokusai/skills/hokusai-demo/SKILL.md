---
name: hokusai-demo
description: Guided end-to-end demonstration of the HOKUSAI BigWaterfall2 (HBW2) plugin — verify configuration, inspect the facility, submit a tiny job, follow it to completion, and read its output. Use when the user wants to try the plugin or confirm it works after install.
---

# HOKUSAI (HBW2) demo

A short, safe walkthrough that exercises the whole plugin against the real
cluster. Explain each step to the user before running it.

## 1. Confirm it's configured

Run `get_facility` — it needs no SSH and returns HBW2's static facts. If it
returns the facility JSON, config parsing works. Then run `get_projects` to
prove SSH + accounting are reachable and to show which projects can be
billed. If either errors with "Plugin not configured", switch to the
**hokusai-configuring** skill.

## 2. Look at live capacity

`get_resources` shows per-partition node occupancy — point out which
partition has idle nodes, i.e. where a job would start soonest.

## 3. Submit a tiny job

Show the user this spec first, then submit it with `submit_job`:

- `executable`: `echo "hello from HBW2"; hostname; sleep 20`
- `resources`: `node_count=1, processes_per_node=1, cpu_cores_per_process=1`
- `environment`: `{"OMP_NUM_THREADS": "1"}`
- `attributes`: `queue_name="mpc", duration="00:03:00"` (account defaults to
  the configured project)

Use `render_job_script` first if the user wants to see the exact sbatch
script.

## 4. Follow it

Poll `get_job_status(job_id)` until the state reaches `completed`
(queued → active → completed). Explain the wait reason if it sits queued.

## 5. Read the output

`read_job_output(job_id)` prints the `slurm-<jobid>.out` log — the user
should see the `hello from HBW2` line and the compute node's hostname.

That covers configuration, facility info, live resources, submission,
monitoring, and output retrieval — the full loop.
