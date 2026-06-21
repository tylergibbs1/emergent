# Grounding the Hunt in Literature (arxiv CLI)

Use the agent-friendly `arxiv` CLI (https://github.com/tylergibbs1/arxivcli) to
ground hypotheses in prior work, find techniques to try, and check whether a
"discovery" is actually already known. No API key required.

## Setup (once, in the Connect Phase)

```bash
git clone https://github.com/tylergibbs1/arxivcli.git && cd arxivcli && bun install && bun link
arxiv schema search    # discover exact args/response shape at runtime
```

If `bun` isn't available locally, run it on the pod, or skip — the loop works
without it, just with less literature grounding.

## When to use it in the loop

- **HYPOTHESIZE step** — before designing an experiment, search for how others
  approached the phenomenon. Pull *ideas to try*, not whole papers.
- **When stuck** (STRATEGIES.md) — scan recent work for unexplored angles.
- **VERIFY step** — when a candidate looks real, search to check it isn't a
  known result or a known artifact. Novelty is part of "revolutionary."

## Context discipline

This is the #1 risk — papers are huge. Always:

- Use `--fields` to shrink responses: `--fields id,title,summary`.
- Use `--max 5` (or less). Read titles + abstracts, not PDFs.
- To digest a specific paper deeply, hand it to an `Agent` subagent and keep only
  the one-paragraph takeaway. Never read a full PDF into the main loop context.

## Useful commands

```bash
# Find approaches to a phenomenon (abstract search, AI category, newest first)
arxiv search "emergent in-context learning" --cat cs.LG --sort submittedDate --order desc --max 5 --fields id,title,summary

# Search by a specific technique you're considering
arxiv search "grokking weight decay" --type abstract --max 5 --fields id,title,summary

# Pull one paper's abstract to decide if it's worth a subagent deep-read
arxiv paper 2201.02177 --fields id,title,summary

# Browse the firehose of a category for unexplored angles
arxiv list cs.LG --max 20 --fields id,title
```

## Turning a paper into an experiment

1. Search → skim abstracts → pick 1-2 concrete, cheap-to-test ideas.
2. Write the idea as a falsifiable hypothesis in the journal, citing the arXiv id.
3. Implement the smallest version that could show the effect on this GPU.
4. Run it through the normal MEASURE → VERIFY pipeline. A paper's claim is a
   hypothesis to test, not a result to assume.
