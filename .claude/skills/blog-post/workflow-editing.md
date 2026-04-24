# Workflow: Editing an Existing Post

Use when: Vincent wants targeted changes to a published or draft post.

## Prior artifacts

If the post went through the outline and writing workflows, these artifacts
may exist and provide useful context:

- **Outline:** `drafts/outlines/<topic>.md` — the originally approved
  structure. Note: the writing workflow freezes the outline and allows the
  writer to deviate, so the published post may not match the outline exactly.
- **Research briefs:** `drafts/briefs/<topic>-transcript-brief.md`,
  `drafts/briefs/<topic>-web-research.md`
- **Critic feedback:** `drafts/critic-feedback/<topic>-outline.md`,
  `drafts/critic-feedback/<topic>-writing.md`

Check for these before starting. They help you understand the post's history
and the reasoning behind its current structure.

## Key files

- **Style guide:** `.claude/docs/writing-style.md`

## Steps

1. **Read the post** — Understand current content, structure, and tone. Skim
   prior artifacts if they exist.
2. **Clarify scope** — If unclear, ask Vincent what specifically needs changing.
3. **Make targeted edits** — Change only what's requested; don't restructure
   or rewrite unless asked.
4. **User review** — Vincent approves changes.

## Optional: style critic loop (up to 3 rounds)

When Vincent asks for it (e.g., "use the critic" or "run it through the
critic"), spawn the style critic as a subagent. This is useful when Vincent
gives a batch of changes and wants you and the critic to work through them
together, presenting the result only when both are satisfied.

**Single point of contact:** Vincent talks to you (the skill leader) only.
All coordination with the critic stays behind the scenes. Don't surface
every inter-agent message to Vincent; surface milestones and questions that
need his input.

Include in the critic's spawn prompt:

- The path to the post being edited
- The style guide path
- What changes were requested
- That the outline (if it exists) is a frozen reference and may not match the
  current post
- **Vincent-mode framing from round 1:** put yourself in Vincent's shoes as
  a reader; the benchmark is the best recent post in the same genre; flag
  structural issues (shape, density, teaser rhythm, metaphor policy), not
  just line-level ones

### Round 1

1. **Spawn critic** — Pass Vincent's feedback (if any) along with the spawn
   prompt. The critic expands on Vincent's feedback in the context of the
   style guide and the post, then appends its full review to
   `drafts/critic-feedback/<topic>-editing.md` (under a `## Round 1`
   heading) and returns a verdict.
2. **You edit** — Apply changes based on the critic's enriched feedback.

### Later rounds

1. **You edit** — Address the critic's feedback from the previous round.
2. **Spawn critic** — The critic is a subagent and doesn't persist between
   rounds. Each spawn, tell it to read the feedback file first so it has
   full context from previous rounds. The critic reviews the post, appends
   feedback (under a `## Round N` heading), and returns a verdict.

The critic's verdict is one of:

- **Ready to present** — edits are good enough for Vincent
- **Present next round** — minor issues, worth one more pass before showing
  Vincent
- **Needs another round** — significant issues require further revision

**Verdict heuristic:** default to "ready to present" when must-fix count ≤3
*and* the draft hasn't changed substantially between rounds. Continuing to
critique past that point invents problems that Vincent's own pass would
absorb cheaply.

The loop exits when the critic says "ready to present" or "present next
round", or after 3 rounds (whichever comes first). Then present the result
to Vincent.
