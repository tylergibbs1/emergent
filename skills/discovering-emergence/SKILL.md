---
name: discovering-emergence
description: >-
  Runs an autonomous ML research loop on a remote RunPod GPU over SSH that
  forms hypotheses, runs experiments, measures multiple capability probes, and
  hunts for emergent properties — capabilities that appear unexpectedly rather
  than being explicitly trained. Keeps reproduced discoveries, discards noise,
  and logs everything to a research journal. Use when the user wants to find
  emergent behavior, run open-ended ML experiments, search for novel or
  revolutionary capabilities, or mentions "find emergent properties",
  "emergence research", "autonomous ML experiments", or provides a RunPod for
  experimentation.
license: MIT
metadata:
  author: tylergibbs
  version: "1.0.0"
  argument-hint: "[emergence.json]"
---

# Discovering Emergence

An autonomous research loop that hunts for **emergent properties** in machine
learning systems: capabilities, behaviors, or phase transitions that arise from
training, scale, or composition but were never explicitly designed. The goal is
to surface genuinely novel phenomena that could advance the field — and to do so
with the skepticism of a real scientist, not a metric-chaser.

Heavy compute runs on a **remote RunPod GPU over SSH**. Your local context window
stays small; the GPU does the work. This is a long-running, mostly autonomous
session — design every step to survive for hours without flooding context.

> Unlike single-metric optimization, emergence is multi-dimensional and you do
> not know in advance what you are looking for. You measure a *battery of probes*
> across conditions and watch for surprise: discontinuities, capabilities absent
> from the objective, sharp generalization, or qualitative shifts. A surprise is
> a candidate, not a discovery. See [references/EMERGENCE.md](references/EMERGENCE.md).

## Discovery Flow

If `emergence.json` exists, skip to [Connect Phase](#connect-phase).

Otherwise:

1. **Get the RunPod connection.** Ask the user for the SSH command RunPod gives
   them (e.g. `ssh root@213.1.2.3 -p 22001 -i ~/.ssh/id_ed25519`) and the
   workspace path on the pod (default `/workspace`). Do not guess credentials.

2. **Explore in parallel.** Launch two `Agent` subagents simultaneously:

   - **Agent 1 — Local project**: scan this repo's structure, language/framework,
     any existing models, training code, datasets, READMEs. Summarize what's here
     and what could be experimented on.
   - **Agent 2 — Remote pod**: SSH in and probe the environment — `nvidia-smi`
     (GPU model, count, VRAM), Python/CUDA/PyTorch versions, installed ML
     packages, disk space, and any code already in the workspace. Report exactly
     what's available. See [references/RUNPOD.md](references/RUNPOD.md).

   Wait for both before proceeding.

3. **Ask one focusing question** (use `AskUserQuestion` if available):

   > I've explored your project and the pod. Here's what's available:
   > - [local summary] / [GPU + env from the pod]
   >
   > **What direction should the emergence hunt take?**
   > e.g., a domain ("small language models", "RL agents", "diffusion"), a
   > phenomenon ("grokking", "in-context learning", "tool use"), or "surprise me
   > within what this GPU can handle."

   Infer everything else: `run_template` from the framework found, `probes` from
   the domain, `budget` from GPU cost, `tag` from today's date.

4. **Write `emergence.json`** — see [references/CONFIG.md](references/CONFIG.md).
   Show it to the user for confirmation, then proceed.

## Connect Phase

1. Parse and validate `emergence.json`. Confirm `budget` is set — **the loop
   spends real money on GPU time**; never run uncapped.
2. SSH to the pod and verify the GPU is live (`nvidia-smi`). Halt on failure.
3. Establish the remote workspace: create `<workspace>/<tag>/`, sync local
   experiment code up to it (see [references/RUNPOD.md](references/RUNPOD.md)),
   and run `setup_check` if configured. Set up the `arxiv` CLI for literature
   grounding (see [references/LITERATURE.md](references/LITERATURE.md)); skip if
   `bun` is unavailable.
4. Locally: create `git checkout -b emergence/<tag>` (append `-2`, `-3`… if it
   exists). Add `results.tsv`, `runs/`, and `*.log` to `.gitignore`.
5. Initialize `results.tsv` (tab-separated header) and `journal.md` (see
   [Journal Format](#journal-format)).
6. **Baseline run**: run the unmodified experiment on the pod, record every probe
   value. This is your reference point for "surprising."
7. Tell the user setup is complete and report the baseline. **This is the last
   interaction.** From here you run autonomously until the budget is spent or the
   user stops you.

## The Hunt Loop

**LOOP until the budget cap is reached. Do NOT stop early. Do NOT ask permission
to continue.** The user may be asleep and expects you to keep hunting.

```
1. ORIENT. Re-read results.tsv and the tail of journal.md for current state and
   open threads. Check elapsed time / spend against budget (see RUNPOD.md).
   If budget exhausted → go to Wind Down.

2. HYPOTHESIZE. Form ONE concrete, falsifiable hypothesis about where emergence
   might appear. Prefer ideas that are:
   - Informed by prior probe readings and journal threads
   - Grounded in prior work — search arXiv for how others approached the
     phenomenon (references/LITERATURE.md); a paper's claim is a hypothesis to
     test, not a result to assume
   - Aimed at a *qualitative* shift, not a 1% metric nudge
   - Cheap enough to fit the remaining budget
   Write the hypothesis to the journal BEFORE running (prevents post-hoc
   storytelling). See references/STRATEGIES.md when out of ideas.

3. DESIGN. Edit ONLY modifiable_files. Change one variable so the result is
   interpretable. Include the full probe battery in the run so you can catch
   surprises you weren't looking for.

4. SYNC & RUN. Push changed files to the pod. Launch the run with ALL output
   redirected to runs/<id>.log on the POD (never stream to your context). Poll
   for completion; kill if it exceeds timeout_seconds.

5. MEASURE. Pull back only the probe summary (grep, not full logs). Record every
   probe value, not just the target. Crash → tail 50 lines; trivial fix → retry
   (max 2-3); broken idea → log as crash and move on.

6. TRIAGE THE RESULT:
   - NOISE / expected → log to results.tsv, note in journal, move on.
   - SURPRISE (discontinuity, capability not trained for, sharp generalization,
     phase transition) → this is a CANDIDATE, not a discovery. Go to step 7.

7. VERIFY before believing. A candidate must survive skepticism:
   - REPRODUCE with a different seed / split. Real emergence repeats.
   - RULE OUT the boring explanation: data leakage, eval bug, memorization,
     metric artifact, prompt giveaway, lucky seed.
   - ABLATE: remove the suspected cause — does the effect vanish as predicted?
   Only a candidate that survives all three is a CONFIRMED finding. Most won't.
   See references/EMERGENCE.md for the full checklist.

8. RECORD. Append a tab-separated row to results.tsv. Write a journal entry:
   hypothesis, what you ran, every probe value, verdict (noise / candidate /
   confirmed / crash), and the next thread it opens.

9. Every 10 experiments, write a SYNTHESIS entry: what patterns are emerging,
   which directions are dead, what to chase next.

10. GOTO 1
```

## Wind Down

When the budget cap is hit (or the user stops the run):

1. Write a final **Findings Report** to `journal.md`: confirmed findings (with
   reproduction evidence), promising-but-unconfirmed candidates, dead ends, and
   the experiments you'd run next.
2. Commit the journal and results. Leave the pod's `runs/` intact for inspection.
3. Stop or pause the RunPod if `budget.stop_pod_on_finish` is set — idle GPUs
   bill by the hour. See [references/RUNPOD.md](references/RUNPOD.md).
4. Surface the report to the user. Do not overclaim: report confirmed findings as
   confirmed and candidates as candidates.

## Critical Rules

- **Protect your context window** — see [references/CONTEXT.md](references/CONTEXT.md).
  This is the #1 risk over a multi-hour run. Redirect all output to logs on the
  pod, grep for probes, only `tail` on crash, read readonly files once.
- **Be your own adversary.** The biggest danger in autonomous discovery is
  fooling yourself. Default to "it's a bug or an artifact" until reproduction and
  ablation prove otherwise. A confirmed null result is more valuable than an
  unverified "breakthrough."
- **Respect the budget.** GPU time costs money every minute. Track spend, honor
  the cap, and never leave the pod running idle.
- **One change per experiment** — so results stay interpretable.
- **The journal is the science.** results.tsv is the numbers; journal.md is the
  reasoning, the surprises, and the verdicts. Both survive git resets.

## Journal Format

`journal.md` is an append-only research log. Each entry:

```
## [exp-id] <one-line hypothesis>   (<timestamp>, <git sha>)
Ran:      <what changed + command>
Probes:   probe_a=… probe_b=… probe_c=…   (baseline: …)
Verdict:  noise | candidate | confirmed | crash
Reason:   <why — including ruled-out boring explanations if a candidate>
Next:     <thread this opens>
```

## Example results.tsv

```
exp_id	git_sha	target_probe	icl_acc	ood_gen	loss	verdict	note
base	a1b2c3d	0.610	0.12	0.20	2.41	keep	baseline
e001	b2c3d4e	0.640	0.13	0.21	2.30	noise	wider MLP, no qualitative shift
e002	c3d4e5f	0.910	0.71	0.22	2.28	candidate	sharp ICL jump at depth 8 — VERIFY
e003	c3d4e5f	0.905	0.70	0.22	2.28	confirmed	reproduced seed 2; ablating depth kills it
e004	d4e5f6g	0.000	0.00	0.00	NaN	crash	OOM at batch 512
```
