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

---

## 2026-04-17 → 2026-04-18 — ZK Proofs Part 2 (Writing)

**Post:** `content/blog/zk-proofs-part2.md` (draft)
**Workflow:** Writing from outline with full team. Seven critic/writer rounds across two days: 2 initial, 3 autonomous "Vincent-mode" rounds, 2 final rounds after Vincent re-engaged.

### Vincent's feedback

1. **"Taking my inline comments and extrapolating to what my comments on the whole doc would have been, guided by the style guide, is the key enabler."** Rounds 3-5 (the autonomous "Vincent-mode" stretch) produced the structural improvements that rounds 1-2 had missed because they were line-level. Preserve and codify this pattern.
2. **Team-lead as the single point of contact works well.** Vincent prefers one teammate to talk to; multi-agent coordination should stay behind the scenes. Should also apply to the editing workflow — update that skill's workflow file.
3. **Worth documenting a "drafting-from-zero" zola checklist.** The `social_media_card` / symlink frictions hit twice this session and could be front-loaded.

### Teammate feedback

1. **Writer: Inline `<!-- FEEDBACK -->` comments from Vincent were the highest-signal input in the loop.** Concrete, in situ, zero ambiguity. Abstract must-fix lists without proposed wording were the lowest signal — the writer had to re-invent the sentence, sometimes wrong on the first pass. Critic's suggested-prose format was the second-best signal.

2. **Writer: Round 3 ("broader rewrite latitude") was the highest-value round.** Rounds 1-2 were line-level polish; round 3 delivered the intro shrink, γ demotion, ρ-subsection flatten, global $A_i \to L_i$ rename, and π-bullets → display-math conversion. One structural round early beats four polish rounds late.

3. **Critic: The first outline-to-draft leap missed Vincent's Obsidian-note mental model entirely.** A phase like "here's Vincent's mental model, write that" could have skipped the round-2 full restructure. The outline critic had signed off on the "one Greek letter per attack" spine that Vincent rejected on first draft.

4. **Critic: Rounds 1-2 were one round too many.** Signal for "ready to present" should flip when must-fix count ≤3 *and* word-count delta between rounds is ~flat. Continuing to critique past that point invents problems.

5. **Critic: "Vincent-mode" framing (rounds 3-5) should be the default framing from round 1.** Channelling Vincent beyond his inline comments = benchmarking against Part 1 on structural dimensions (section count, paragraph density, teaser rhythm, metaphor policy), not just voice. Give the critic permission to flag structural issues from the start.

6. **Writer: γ-as-`<details>` got revisited three times across rounds.** Task briefs should name "Vincent-only decisions" explicitly so the writer doesn't keep changing state across rounds.

7. **Writer: Task re-delivery notifications.** Around 7 ghost assignments for already-completed tasks. Delivery-layer issue, not workflow, but writer should silently verify state rather than acknowledging each.

### Patterns to address

- **Push structural review earlier.** The outline-approval bar should include "does this match the structural shape Vincent wants?", not just completeness. If the outline came from a Vincent-authored artifact (Obsidian note, transcript), that artifact's structure is the spine — test against it explicitly.
- **Make "Vincent-mode" the critic's default framing.** Brief every round as: "Put yourself in Vincent's shoes as a reader; Part 1 is the benchmark; flag structural issues, not just line-level ones." Move it from a round-3 unlock to the round-1 baseline.
- **Verdict heuristic:** must-fix ≤3 + word count stable across rounds → "ready to present." Stop hunting for polish items.
- **Vincent-only decisions get explicit names in task briefs.** Add a standing "Don't touch" list per round so the writer doesn't re-open decisions Vincent has scoped out.
- **Front-load style-guide learnings into the writer's first-draft brief.** Session-specific style traps (metaphor policy, forbidden openers, voice calibration) should come from `writing-style.md` directly, not be rediscovered in round 3.
- **Zola drafting checklist.** Document in the blog-post skill:
  - Omit `social_media_card` from frontmatter until the referenced image file exists (Tabi's `page.html` throws on missing files, breaking `zola serve --drafts`).
  - Omit the leading `![hero](...)` markdown if the banner image doesn't exist yet.
  - Draft file lives at `content/blog/<slug>.md` with `draft = true` in frontmatter, NOT in `drafts/posts/` via symlink — Zola's file watcher doesn't follow symlinks for hot-reload.

### Running notes (kept for continuity)

- **Don't include `social_media_card` in frontmatter until the image actually exists.** Tabi theme's `page.html` calls a `throw` that fails the build when the referenced file is missing; `zola serve --drafts` errors out and the draft can't be previewed.
