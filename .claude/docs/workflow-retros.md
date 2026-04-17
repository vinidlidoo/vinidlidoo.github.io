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

---

## 2026-04-17 — ZK Proofs Part 2 (Outline)

**Post:** `drafts/outlines/zk-proofs-part2.md`
**Workflow:** Outline with full team (transcript-researcher, web-researcher, outline-writer, style-critic). 2 critic rounds.

### Teammate feedback

1. **Verify load-bearing examples before the first draft, not after the critic catches the gap.** The §5 `a × a = out` attack was the post's most important concrete example. The web researcher's $C_a \equiv 0$ insight (which made the arithmetic airtight) came only in round 1 verification, after the critic flagged the hand-wave. If the web researcher had been asked to verify key technical examples during Phase 1, the outline writer could have used clean arithmetic from the start, and the critic's round 1 verdict might have been "ready to present" instead of "present next round." (style-critic, outline-writer)

2. **Batch verification with initial research.** The round-1 critic claims overlapped heavily with the web researcher's initial scope. Including "stretch goal" verification items (anticipated critic claims) in the Phase 1 web research task would avoid a second round of overlapping web searches. (web-researcher)

3. **Pre-download inaccessible PDFs.** WebFetch returns 403 on eprint.iacr.org PDFs, which are primary sources for cryptography posts. Having Vincent pre-download key papers (Gabizon 2019, Groth 2016) into `drafts/references/` would let the web researcher cite exact formulas instead of triangulating from secondary sources. (web-researcher)

4. **Transcript researcher needs Write tool.** The transcript-researcher agent type is read-only and cannot write files, so the brief had to be delivered as an inline message for the leader to save manually. This added friction and context overhead. (transcript-researcher)

5. **Flag prior briefs as required reading in transcript tasks.** The earlier transcript brief (`zk-proofs-transcript-brief.md`) was essential for knowing what was already covered vs. new material. Listing it in the task description upfront would save the researcher from discovering the dependency mid-work. (transcript-researcher)

6. **Avoid duplicate task dispatches.** The style critic received duplicate round 1 and round 2 assignments (leader re-dispatched after tasks were already complete). Check task status before re-dispatching to avoid wasted context. (style-critic)

7. **Single source of truth for critic feedback.** The revision task description duplicated every item from the critic feedback file. Leaner approach: task description says "apply critic feedback at [path]" and the file is the single source of truth. (outline-writer)

### Patterns to address

- The "verify load-bearing examples early" pattern (item 1) is the highest-leverage improvement. Consider adding a Phase 1.5 step: after Phase 1 research but before Phase 2 outlining, have the web researcher verify any concrete attack/example the outline is known to depend on.
- Pre-downloading inaccessible primary sources (item 3) is a Vincent-side action. Flag at the start of future crypto-domain sessions.
- Duplicate dispatches (item 6) are a leader coordination issue. Use TaskGet to check task status before sending messages.
