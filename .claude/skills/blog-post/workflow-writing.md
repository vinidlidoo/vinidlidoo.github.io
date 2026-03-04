# Workflow: Writing from Outline

Use when: User has an approved outline in `drafts/outlines/` and wants the
full post written.

**Guiding principle:** The outline is a starting point, not a contract. The
primary goal is an engaging, well-written post that matches Vincent's voice and
style. If the outline's structure makes content heavy, hard to digest, or
misses opportunities for better flow, the writer and critic should deviate and
explain why. Voice and style matching (per the style guide and recent posts) is
paramount.

The outline file itself stays frozen as a record of what was approved.
Structural deviations are documented in the critic feedback file with
rationale.

**Agents:** Up to 3 teammates, orchestrated by the skill leader (the main
agent running this skill). All agents persist through Phase 3 so that the
writer and critic can message the web researcher for clarifications.

| Agent | Type | Objective | Output |
|-------|------|-----------|--------|
| **Writer** | Ad-hoc `general-purpose` agent | Write the post section by section; revise based on critique and user feedback. May propose structural deviations from the outline when the prose demands it. | `drafts/posts/<topic>.md` |
| **Style critic** | `style-critic` agent | Review for style alignment, flow, and technical soundness; produce a verdict. Should flag when outline structure hurts the post and recommend adjustments. | `drafts/critic-feedback/<topic>-writing.md` (appended each round) |
| **Web researcher** | Ad-hoc `general-purpose` agent | Verify technical claims, key numbers, and completeness; update research brief across phases. | `drafts/briefs/<topic>-web-research.md` (updated across phases) |

**Key files:**

- **Style guide:** `.claude/docs/writing-style.md`
- **Outline-phase artifacts:** `drafts/outlines/<topic>.md`,
  `drafts/briefs/<topic>-transcript-brief.md`,
  `drafts/briefs/<topic>-web-research.md`,
  `drafts/critic-feedback/<topic>-outline.md`

## Coordination

**Task list as backbone:** The team shares a task list (created with
`TeamCreate` in Phase 1). The leader creates tasks, teammates complete them,
and the leader monitors via `TaskList`. Use `addBlockedBy` in `TaskUpdate`
to enforce ordering between tasks — described as "depends on" below.

**Spawn prompts:** When spawning the writer and critic, include the guiding
principle (at the top of this file) in their spawn prompts so they are aware
of it throughout.

**Direct messaging:** Agents use `SendMessage` for lightweight
notifications and clarification questions between teammates.

**Consulting Vincent:** Check in at the end of Phase 1. Do not create
Phase 2 tasks until Vincent has responded. Escalate mid-phase blockers to
Vincent via `AskUserQuestion`.

## Phase 1: Writing

Create the team with `TeamCreate`, then create and assign:

- **Task: Write first draft** → assign to writer. Provide paths to all
  outline-phase artifacts, style guide, and 2-3 recent posts for voice
  calibration. The writer absorbs these, then writes the post section by
  section following the outline but adapting structure where the prose calls
  for it. Before marking the task complete, the writer self-reviews: check
  voice consistency and flag any deviations from the outline with rationale.

**Check-in with Vincent:** Present the first draft and ask if the direction
is right. Wait for Vincent's response before creating Phase 2 tasks.

## Phase 2: Critique-revise loop (up to 3 rounds)

Spawn the **style critic**. Each round, create all tasks upfront with
dependencies to enforce ordering:

- **Task: Review draft (round N)** → style critic. In round 1, depends on
  the Phase 1 writing task. In later rounds, depends on the previous round's
  revise task. The critic produces:
  1. **Style feedback** → appends to `drafts/critic-feedback/<topic>-writing.md`
     (under a `## Round N` heading)
  2. **Claims list** → messages the web researcher with a summary of claims
     to verify (also contained in the feedback file)
  3. **Verdict** → messages the skill leader
- **Task: Verify claims (round N)** → web researcher (if claims were flagged
  by the critic). Depends on review task. Updates the research brief.
- **Task: Revise draft (round N)** → writer. In round 1, include both
  Vincent's feedback from the Phase 1 check-in and the critic's style
  feedback. In later rounds, use the critic's style feedback. Depends on
  review task, and verify claims task if it exists.

The loop exits when the critic says "ready to present" or "present next round",
or after 3 rounds (whichever comes first).

## Phase 3: Present to Vincent

Deliver the draft with a summary of (using
`drafts/critic-feedback/<topic>-writing.md` as the source):

- What the critic flagged across rounds
- What was addressed and what remains open
- Any structural deviations from the original outline, with rationale

Shut down all teammates and clean up the team with `TeamDelete`.

## Phase 4: Iterate

Address Vincent's feedback until approved. This is lighter — just the leader
directly editing, no team needed.

## Phase 5: Finalize

Create final post at `content/blog/slug-matching-title.md`.
