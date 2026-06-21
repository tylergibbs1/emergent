# emergent

An Agent Skill for Claude Code that runs an **autonomous ML research loop** to
hunt for **emergent properties** — capabilities that arise from training, scale,
or composition but were never explicitly designed. Heavy compute runs on a remote
**RunPod GPU over SSH**; your context window stays small while the GPU does the work.

The goal is to surface genuinely novel phenomena with the skepticism of a real
scientist: every surprise is a *candidate*, not a discovery, until it survives
reproduction, ablation, and a hunt for the boring explanation.

## What it does

1. Explores your local project and the RunPod environment in parallel.
2. Writes an `emergence.json` config (probes, run command, budget cap).
3. Runs a baseline, then loops: **orient → hypothesize → run → measure → verify**.
4. Grounds hypotheses in arXiv literature via the [`arxiv` CLI](https://github.com/tylergibbs1/arxivcli).
5. Keeps reproduced findings, discards noise, logs everything to a research journal.
6. Respects a spend/time budget and stops the pod when done.

## Install

```bash
npx skills add tylergibbs1/emergent
```

Or install manually:

```bash
cp -r skills/discovering-emergence ~/.claude/skills/discovering-emergence
```

Or clone directly into your skills directory:

```bash
git clone https://github.com/tylergibbs1/emergent.git \
  ~/.claude/skills/emergent
```

Then in Claude Code:

```
find emergent properties — here's my runpod: ssh root@<ip> -p <port> -i ~/.ssh/id_ed25519
```

The skill triggers on phrases like "find emergent properties", "emergence
research", "autonomous ML experiments", or when you provide a RunPod for
experimentation.

## Layout

```
skills/discovering-emergence/
├── SKILL.md                  # main loop and workflow
├── references/
│   ├── EMERGENCE.md          # what counts as emergence + verification checklist
│   ├── RUNPOD.md             # SSH ops, remote runs, budget tracking, pod shutdown
│   ├── STRATEGIES.md         # where to look when out of ideas
│   ├── LITERATURE.md         # grounding hypotheses with the arxiv CLI
│   ├── CONTEXT.md            # context-window discipline for long runs
│   └── CONFIG.md             # emergence.json field reference
└── examples/
    └── emergence.json        # sample config
```

## Credits

Loop pattern adapted from [autoresearchskill](https://github.com/tylergibbs1/autoresearchskill)
(inspired by [Karpathy's autoresearch](https://github.com/karpathy/autoresearch)).
Literature search via [arxivcli](https://github.com/tylergibbs1/arxivcli).
Plugin format follows [linearprincipals](https://github.com/tylergibbs1/linearprincipals).
