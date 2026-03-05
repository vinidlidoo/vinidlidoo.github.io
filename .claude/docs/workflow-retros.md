# Workflow Retrospectives

Teammate feedback collected at the end of writing sessions. Periodically
review for patterns and fold into the workflow files.

---

## 2026-03-04 — ZK Proofs Part 1 (Editing)

**Post:** `content/blog/zk-computation-to-polynomials.md`
**Workflow:** Editing with critic loop (6+ critic rounds across 2 context windows)

### Vincent's feedback

1. **Editor needs more independent critical thinking.** Too many rounds where Vincent had to point out issues (e.g., presenting incomplete protocol as working, the prover-not-bound-to-circuit gap) that the editor should have caught. The editor should spend more time reasoning about whether the draft's claims actually hold before committing edits, rather than applying surface-level changes and waiting for Vincent to poke holes.

2. **Too many critic rounds for too little progress.** 6+ rounds across two context windows felt excessive. The critic loop needs a way to achieve more per round. Possible improvements: the critic should flag deeper conceptual issues (not just style), and the editor should resolve more problems in a single pass rather than iterating incrementally.

### Patterns to address

- The editor tends to take feedback literally without asking "does this claim actually make sense?" Would benefit from a self-review step before presenting to Vincent: re-read the section as a skeptical reader and check whether the logic holds.
- The critic focuses heavily on style and phrasing but misses conceptual gaps (e.g., never flagged that the "verifier checks at random τ" framing was misleading without protocol machinery). Consider adding a "technical soundness" pass to the critic's mandate.
- Context window pressure leads to bulk-applying feedback without enough thought. When many items queue up, prioritize the conceptual ones over cosmetic fixes.
- The critic mostly expanded on Vincent's points rather than finding independent issues. Consider running the critic *before* Vincent reviews, so it catches surface issues and Vincent's time is spent on harder conceptual problems.
- When a section resists multiple rounds of editing, that's a signal the problem is conceptual, not stylistic. Step back and ask "what has the post actually established?" before rewriting prose.
