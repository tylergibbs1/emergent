# Hunting Strategies (Where to Look for Emergence)

Emergence hides at boundaries and transitions. When you need a new hypothesis,
work down this list. Always pre-register the hypothesis in the journal before
running, and change one variable at a time.

## Sweep the axes emergence lives on

1. **Scale sweep** — vary model width/depth or parameter count across several
   points. Plot a probe vs. scale. You're hunting for a *knee*, not a slope.
2. **Training-time sweep** — keep training well past convergence on the train set.
   Grokking and late circuit formation appear long after the loss flattens.
3. **Data sweep** — vary dataset size, diversity, or composition. Some skills
   only switch on past a data threshold or with a specific mixture.
4. **Compositionality** — train on primitives, test on novel combinations.

## Change the structure

5. **Architecture deltas** — attention vs. none, depth vs. width at fixed params,
   normalization, weight tying, recurrence. Watch off-objective probes.
6. **Objective deltas** — auxiliary losses, curriculum, self-distillation,
   multi-task heads. New objectives can unlock unrelated capabilities.
7. **Initialization / dynamics** — LR schedule, warmup, weight decay. Phase
   transitions are often a training-dynamics phenomenon.

## Read the model, not just the metric

8. **Inspect internals** — probe for induced circuits, sparse features,
   representation rank, neuron specialization. Structure often forms *before* the
   behavioral metric moves.
9. **Read actual samples** — pull a handful of raw outputs. Qualitative shifts
   (a new strategy, a new failure mode) are invisible to scalar metrics.

## Use the history

10. **Mine the journal** — which threads spiked then died? Re-run with verification.
11. **Search the literature** — `arxiv search` for the phenomenon or a technique
    you haven't tried (references/LITERATURE.md). Turn an abstract into the
    smallest testable experiment.
12. **Combine near-misses** — two changes that each almost did something; try both.
13. **Try the opposite / vary the dose** — if more X did nothing, try far more, or
    less. Emergence is often a threshold, so small steps miss it.

## Go radical (when the budget allows)

14. **Different paradigm** — swap the learning rule, the data modality, or the
    task framing entirely. The biggest surprises come from the least-explored
    corner, but these runs are expensive — gate them on remaining budget.

## When everything is flat

- Widen the probe battery: you may be measuring the wrong thing.
- Increase resolution on a promising axis (more points around a suspected knee).
- Simplify: strip the system to its core and rebuild — emergence is easier to see
  in a minimal setup than a kitchen-sink one.
