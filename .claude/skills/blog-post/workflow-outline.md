# Workflow: Creating an Outline

Use when: Vincent asks for a blog post outline.

**Expected inputs** (any combination):

- A conversation transcript (Claude conversation where Vincent explored the topic)
- A topic description or brief
- Specific references or links to include

**Agents:** Up to 4 teammates, orchestrated by the skill leader (the main
agent running this skill). All agents persist through Phase 3 so that the
critic and outline writer can message any teammate for clarifications (e.g.,
asking the transcript researcher about Vincent's interests, or the web
researcher about a specific claim).

| Agent | Type | Objective | Output |
|-------|------|-----------|--------|
| **Transcript researcher** | `transcript-researcher` agent | Parse transcript for Vincent's interests, confusion points, and engagement patterns; produce a prioritized brief. | `drafts/briefs/<topic>-transcript-brief.md` |
| **Web researcher** | Ad-hoc `general-purpose` agent | Verify technical claims, key numbers, and completeness; update research brief across phases. | `drafts/briefs/<topic>-web-research.md` (updated across phases) |
| **Outline writer** | Ad-hoc `general-purpose` agent | Synthesize research into a draft outline; revise based on critique and user decisions. | `drafts/outlines/<topic>.md` |
| **Style critic** | `style-critic` agent | Review for style alignment, pedagogical flow, and technical soundness; produce a verdict. | `drafts/critic-feedback/<topic>-outline.md` (appended each round) |

**Key files:**

- **Style guide:** `.claude/docs/writing-style.md`
- **Outline template:** `.claude/skills/blog-post/outline-template.md`

## Coordination

**Task list as backbone:** The team shares a task list (created with
`TeamCreate` in Phase 1). The leader creates tasks, teammates complete them,
and the leader monitors via `TaskList`. Use `addBlockedBy` in `TaskUpdate`
to enforce ordering between tasks — described as "depends on" below.

**Direct messaging:** Agents use `SendMessage` for lightweight
notifications and clarification questions between teammates.

**Consulting Vincent:** Check in at the end of Phase 1 and Phase 2 (see each
phase for specifics). Escalate mid-phase blockers to Vincent via
`AskUserQuestion`.

## Phase 1: Research (parallel)

Create the team with `TeamCreate`, then create and assign tasks:

- **Task: Parse transcript** → assign to transcript researcher. Give it the
  high-level topic and the transcript path. It writes a prioritized brief and
  marks the task complete. Stays on standby after. **Skip if no transcript.**
- **Task: Web research** → assign to web researcher. Give it the topic, any
  user-provided references, and a summary of what the post will cover. It
  writes an annotated research brief and marks the task complete.
- **Obsidian lookup** (skill leader, no task needed) — Search for relevant
  notes in Vincent's Obsidian vault `Study/` folder using the Obsidian skill.

**Check-in with Vincent:** Summarize what the researchers found, what the
Obsidian notes contain (if any), and flag open questions.

## Phase 2: Outline writing

Create and assign:

- **Task: Write first draft outline** → assign to outline writer. Provide it
  with paths to transcript brief, web research brief, Obsidian note content,
  style guide, outline template, and memory files. Before marking the task
  complete, the writer self-reviews: verify all user decisions from the spawn
  prompt are applied (terminology, scope, weight) and cross-reference concrete
  numbers against the web research brief.

**Check-in with Vincent:** Present the first draft outline and ask if the
direction is right before entering the critique-revise loop.

## Phase 3: Critique-revise loop (up to 3 rounds)

Spawn the **style critic**. Each round, create all tasks upfront with
dependencies to enforce ordering:

- **Task: Review outline (round N)** → style critic. In round 1, depends
  on the Phase 2 outline task. In later rounds, depends on the previous
  round's revise task. The critic produces:
  1. **Style feedback** → appends to `drafts/critic-feedback/<topic>-outline.md`
     (under a `## Round N` heading)
  2. **Claims list** → messages the web researcher with a summary of
     claims to verify (also contained in the feedback file)
  3. **Verdict** → messages the skill leader
- **Task: Verify claims (round N)** → web researcher (if claims were
  flagged by the critic). Depends on review task. Updates the research
  brief.
- **Task: Revise outline (round N)** → outline writer. In round 1,
  include both Vincent's feedback from the Phase 2 check-in and the
  critic's style feedback. In later rounds, use the critic's style
  feedback. Depends on review task, and verify claims task if it exists.

The loop exits when the critic says "ready to present" or "present next round",
or after 3 rounds (whichever comes first).

## Phase 4: Present to user

Deliver the final outline with a summary of (using
`drafts/critic-feedback/<topic>-outline.md` as the source):

- What the critic flagged across rounds
- What was addressed and what remains open
- Any unverified claims or gaps

Shut down all teammates and clean up the team with `TeamDelete`.
