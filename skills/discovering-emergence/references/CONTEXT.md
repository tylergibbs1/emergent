# Context Window Management

This loop may run for hours across hundreds of experiments. If experiment output
fills your context, the session dies mid-hunt. Treat context as the scarcest
resource after GPU budget.

## Rules

1. **Output lives on the pod.** Every run redirects to `runs/<id>.log 2>&1` on the
   RunPod. Never stream training output into your context.
2. **Grep for probes, never read logs.** Pull only `grep "^probe_" runs/<id>.log`.
   The full log stays remote.
3. **Tail only on crash.** `tail -n 50 runs/<id>.log` for diagnosis, nothing more.
4. **Read readonly files once.** They don't change; re-reading wastes context.
5. **results.tsv + journal.md are your memory.** They survive git resets and hold
   the full history. Re-read their tails to orient instead of recalling from
   context.
6. **Sync diffs, not trees.** Push only changed files each iteration; don't cat
   large files back and forth.
7. **Short commits and journal entries.** One line of reasoning per field, not
   paragraphs.
8. **Subagents for bursty reading.** If you must digest a large file, a paper, or
   a noisy log, delegate it to an `Agent` and keep only the summary.
