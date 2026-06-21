# emergence.json Reference

## Required Fields

| Field | Description |
|-------|-------------|
| `tag` | Run name; used for branch `emergence/<tag>` and pod dir `<workspace>/<tag>` |
| `runpod.ssh` | Full SSH command RunPod provides (direct TCP preferred) |
| `runpod.workspace` | Base path on the pod (e.g. `/workspace`) |
| `modifiable_files` | Files the agent may edit (glob patterns OK) |
| `readonly_files` | Context files the agent must NOT edit |
| `run_template` | Command to launch one experiment on the pod |
| `probes` | List of probe specs (see below) — the measurement battery |
| `budget` | Spend/time cap (see below) — REQUIRED; the loop never runs uncapped |

## Probe spec

Each probe prints one greppable line `probe_<name>: <value>`:

```json
{ "name": "icl_acc", "grep": "^probe_icl_acc:", "kind": "off-objective", "note": "in-context learning eval" }
```

`kind` is one of `target`, `off-objective`, `generalization`, `internal`, `sanity`.
Always include at least one `off-objective` probe — that's where emergence hides.

## budget

```json
{ "max_hours": 12, "max_usd": 60, "usd_per_hour": 0.69, "stop_pod_on_finish": true }
```

| Field | Description |
|-------|-------------|
| `max_hours` | Wall-clock cap since setup. Hit it → Wind Down. |
| `max_usd` | Spend cap. Hit it → Wind Down. |
| `usd_per_hour` | The pod's hourly rate, for spend estimation. |
| `stop_pod_on_finish` | Stop the pod at Wind Down to halt billing (needs API/runpodctl). |

## Optional Fields

| Field | Default | Description |
|-------|---------|-------------|
| `sync_command` | rsync template | Override how local files reach the pod (e.g. git pull) |
| `setup_check` | — | Command run once to verify prerequisites on the pod |
| `timeout_seconds` | `1800` | Max wall-clock per experiment run |
| `seeds` | `[0, 1]` | Seeds used for the reproduction step in verification |
| `no_new_dependencies` | `true` | Prevent installing packages on the pod |
| `domain` | — | Free-text description of the research direction |

## Example

```json
{
  "tag": "jun21",
  "runpod": {
    "ssh": "ssh root@213.1.2.3 -p 22001 -i ~/.ssh/id_ed25519",
    "workspace": "/workspace"
  },
  "modifiable_files": ["experiment.py"],
  "readonly_files": ["model.py", "data.py", "README.md"],
  "run_template": "python experiment.py --config run.yaml",
  "probes": [
    { "name": "target_acc", "grep": "^probe_target_acc:", "kind": "target" },
    { "name": "icl_acc",    "grep": "^probe_icl_acc:",    "kind": "off-objective" },
    { "name": "ood_gen",    "grep": "^probe_ood_gen:",    "kind": "generalization" },
    { "name": "feat_sparsity", "grep": "^probe_feat_sparsity:", "kind": "internal" },
    { "name": "train_loss", "grep": "^probe_train_loss:", "kind": "sanity" }
  ],
  "budget": { "max_hours": 12, "max_usd": 60, "usd_per_hour": 0.69, "stop_pod_on_finish": true },
  "timeout_seconds": 1800,
  "seeds": [0, 1, 2],
  "domain": "emergent in-context learning in <10M-param transformers"
}
```

## Writing good configs

- **Every probe must print a greppable line.** If the experiment doesn't print
  `probe_<name>: <value>`, add the print to a modifiable file first.
- **Always include an off-objective probe** — a capability the loss doesn't
  reward. Emergence rarely shows up in the thing you're optimizing.
- **Set the budget honestly.** Look up the pod's real hourly rate; an uncapped
  loop on a rented GPU is a way to wake up to a large bill.
- **Scope modifiable_files tightly** — fewer files = more interpretable changes.
- **Size experiments to the GPU** — short enough that you get dozens of runs
  inside the budget, not two.
