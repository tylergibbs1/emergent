# Running on a RunPod GPU over SSH

## Contents
- Connecting
- Probing the environment
- Syncing code to the pod
- Running experiments without flooding context
- Tracking spend and the budget cap
- Stopping the pod

## Connecting

RunPod gives the user an SSH command on the pod's Connect tab. Store the whole
thing in `emergence.json` as `runpod.ssh` and define a shell alias for reuse:

```bash
POD="ssh root@213.1.2.3 -p 22001 -i ~/.ssh/id_ed25519 -o StrictHostKeyChecking=accept-new"
$POD 'echo connected && hostname'
```

Two SSH styles RunPod exposes:
- **Direct TCP** (`root@<ip> -p <port>`) — supports `rsync`/`scp`, preferred.
- **Basic proxy** (`<podid>@ssh.runpod.io`) — terminal only, no file transfer.
  If only the proxy is available, sync via `git` (push from local, pull on pod)
  or `runpodctl send/receive`.

Always test the connection in the Connect Phase and halt on failure rather than
looping against a dead pod.

## Probing the environment

Run once, at setup, and record the results so you don't re-query:

```bash
$POD 'nvidia-smi --query-gpu=name,memory.total,memory.used --format=csv'
$POD 'python -c "import torch; print(torch.__version__, torch.cuda.is_available(), torch.cuda.device_count())"'
$POD 'nproc && free -g && df -h /workspace'
```

Let the actual VRAM and GPU count shape experiment sizing — do not assume an A100
if it's a 4090.

## Syncing code to the pod

Direct TCP — use rsync (fast, incremental, skips junk):

```bash
rsync -az -e "ssh -p 22001 -i ~/.ssh/id_ed25519" \
  --exclude '.git' --exclude 'runs' --exclude '__pycache__' \
  ./ root@213.1.2.3:/workspace/<tag>/
```

Proxy-only — commit locally and `git pull` on the pod each iteration. Keep a
helper so each loop iteration is one cheap command.

**Sync only the modifiable files each iteration** after setup — the readonly
context and datasets are already up there. Datasets stay on the pod; never pull
them down.

## Running experiments without flooding context

The cardinal rule: experiment output lives in a log **on the pod**, never in your
context. Pattern:

```bash
# Launch detached so a dropped SSH session doesn't kill the run
$POD 'cd /workspace/<tag> && nohup python experiment.py > runs/<id>.log 2>&1 & echo $!'
# ... poll for completion (PID gone or a DONE sentinel in the log) ...
$POD 'cd /workspace/<tag> && grep -E "^probe_" runs/<id>.log'   # pull ONLY probes
```

- Enforce `timeout_seconds`: if exceeded, `$POD 'kill <pid>'`, log a timeout,
  move on.
- On crash, pull only `tail -n 50 runs/<id>.log` for diagnosis — never the whole
  log.
- Checkpoint long trainings to `/workspace` so a pod restart doesn't lose work.
- Use `nohup`/`tmux` so the run survives transient SSH drops over a long session.

## Tracking spend and the budget cap

GPU time bills by the minute whether or not you're using it. Before each loop
iteration, check the cap from `emergence.json` `budget`:

```bash
# Wall clock since setup — compare against budget.max_hours
$POD 'cat /workspace/<tag>/.started 2>/dev/null || date +%s | tee /workspace/<tag>/.started'
```

Estimate spend as `elapsed_hours * budget.usd_per_hour`. When `max_hours` or
`max_usd` is reached, go to **Wind Down** — do not start another experiment.

## Stopping the pod

If `budget.stop_pod_on_finish` is true, stop the pod at Wind Down so it stops
billing. From the pod you cannot stop yourself reliably; prefer the RunPod API
from local if a key is configured:

```bash
runpodctl stop pod <POD_ID>     # if runpodctl + RUNPOD_API_KEY available locally
```

If no API access is configured, do NOT assume the pod stopped — tell the user
clearly in the Findings Report that the pod is still running and must be stopped
manually to avoid charges.
