# Detecting Emergence

## Contents
- What counts as emergence
- Signals worth chasing
- The verification checklist
- How to fool yourself (and how not to)
- Probes: building the battery

## What counts as emergence

An **emergent property** is a capability or behavior that arises from a system
but is not present in, or simply predictable from, its parts or training
objective. In ML this shows up as:

- **Capability jumps** — performance on a task is flat (near chance) and then
  rises sharply with scale, training time, or a structural change.
- **Capabilities outside the objective** — the model does something it was never
  trained to do (in-context learning, arithmetic from text, tool use, planning).
- **Phase transitions** — grokking (sudden generalization long after fitting the
  train set), sudden formation of internal circuits, order/disorder shifts.
- **Compositional generalization** — correct behavior on combinations never seen
  in training.
- **Self-organization** — structure (induction heads, sparse features, modular
  subnetworks) forming without being imposed.

Emergence is a *claim about surprise relative to a baseline*. Without a baseline
and an expectation, you cannot tell emergence from ordinary improvement. Always
measure against the baseline from the Connect Phase.

## Signals worth chasing

Watch the probe battery for these, not just the headline metric:

| Signal | What you'd see in the probes |
|--------|------------------------------|
| Discontinuity | A probe steps up sharply across a small change in scale/steps/depth |
| Off-objective skill | A capability probe rises while the training loss is unremarkable |
| Sharp generalization | OOD/held-out probe jumps while train metric was already saturated |
| Threshold | Effect is absent below some size/data/step count, present above |
| Qualitative shift | Sample outputs change in kind, not degree (read actual samples) |

A flat 1-2% nudge on the target metric is **not** a signal. Hold out for jumps,
thresholds, and kind-changes.

## The verification checklist

A surprise is a **candidate**. Promote it to **confirmed** only if it survives
all of this. Run it before writing anything down as a finding.

```
Candidate verification:
- [ ] REPRODUCE: re-run with a new seed and a fresh data split. Effect persists?
- [ ] LEAKAGE: is any eval data reachable from training data? Check overlap.
- [ ] MEMORIZATION: does it hold on freshly generated / never-seen inputs?
- [ ] METRIC ARTIFACT: is the metric measuring what you think? Inspect raw samples.
- [ ] PROMPT GIVEAWAY: does the eval prompt leak the answer or the format?
- [ ] ABLATION: remove the suspected cause — the effect should disappear. Does it?
- [ ] DOSE-RESPONSE: vary the cause's magnitude — does the effect scale sensibly?
```

If any boring explanation survives, it's not emergence — it's a bug or an
artifact, and that is a useful result to log honestly.

## How to fool yourself (and how not to)

Autonomous discovery is a minefield of self-deception. The named traps:

- **Garden of forking paths** — running many probes and reporting the one that
  spiked. Fix: pre-register the hypothesis in the journal *before* the run; treat
  unexpected spikes as new hypotheses to test fresh, not as findings.
- **Seed lottery** — one lucky seed looks magical. Fix: reproduce on ≥2 seeds.
- **Eval bug as discovery** — a broken metric reads as superhuman. Fix: inspect
  raw outputs by hand for any candidate.
- **Confirmation drift** — tuning the eval until the result appears. Fix: freeze
  the eval before the run; changing it invalidates the candidate.
- **Scale confound** — "bigger is better" is expected, not emergent, unless it's
  *discontinuous*. Fix: require a threshold/jump, not a smooth gain.

Default posture: **assume artifact until proven otherwise.** A confirmed null is
worth more than an unverified breakthrough.

## Probes: building the battery

Probes are cheap, automated measurements run every experiment. Aim for 3-6 that
each print one greppable line: `probe_<name>: <value>`.

Good probes for an emergence hunt:
- **Target probe** — the capability the current thread is about.
- **Off-objective probes** — capabilities the training does NOT directly reward
  (these are where emergence hides).
- **Generalization probe** — held-out / OOD / compositional split.
- **Internal probe** (optional) — a structural measure: feature sparsity, circuit
  presence, representation rank, weight-norm dynamics.
- **Sanity probe** — train loss / fit, to distinguish "learned nothing" from
  "learned but doesn't generalize."

Keep probes fixed across the run so values are comparable to the baseline. If you
must add a probe mid-hunt, re-run the baseline with it so comparisons stay valid.
