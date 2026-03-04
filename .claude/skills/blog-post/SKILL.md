---
name: blog-post
description: Create blog post outlines and drafts for Vincent's Zola blog. Orchestrates agent teams for research, outline creation, and style review. Use when asked to create a new outline, write or edit a blog post, or help with blog content.
---

# Blog Post Skill

Orchestrate blog post creation using agent teams. The primary workflow spins up parallel research agents, synthesizes their output into an outline, and runs a critique-revise loop before presenting to the user.

For long/complex topics, propose splitting into multiple posts before outlining.

## Workflows

### Workflow 1: Creating an Outline

Use when: Vincent asks for a blog post outline.

**Expected inputs** (any combination):

- A conversation transcript (Claude conversation where Vincent explored the topic)
- A topic description or brief
- Specific references or links to include

**Agents:** Up to 4 teammates, orchestrated by the skill leader (the main
agent running this skill). All agents persist through phase 3 (the transcript
researcher stays on standby to answer questions about Vincent's interests).

| Agent | Type | Objective | Output |
|-------|------|-----------|--------|
| **Transcript researcher** | `transcript-researcher` agent | Parse transcript for Vincent's interests, confusion points, and engagement patterns; produce a prioritized brief. | `drafts/briefs/<topic>-transcript-brief.md` |
| **Web researcher** | Ad-hoc `general-purpose` agent | Verify technical claims, key numbers, and completeness; update research brief across phases. | `drafts/briefs/<topic>-web-research.md` (updated across phases) |
| **Outline writer** | Ad-hoc `general-purpose` agent | Synthesize research into a draft outline; revise based on critique and user decisions. | `drafts/outlines/<topic>.md` |
| **Style critic** | `style-critic` agent | Review for style alignment, pedagogical flow, and technical soundness; produce a verdict. | Feedback via `SendMessage` |

#### Coordination

**Task list as backbone:** The team shares a task list (created with
`TeamCreate` in Phase 1). The leader creates tasks, teammates complete them,
and the leader monitors via `TaskList`.

**Direct messaging:** Agents use `SendMessage` for ephemeral per-round
feedback (e.g., style critique results, claim verification questions).

**Consulting Vincent:** Check in at the end of Phase 1 and Phase 2 (see each
phase for specifics). Escalate mid-phase blockers to Vincent via
`AskUserQuestion`.

#### Phase 1: Research (parallel)

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

#### Phase 2: Outline writing

Create and assign:

- **Task: Write first draft outline** → assign to outline writer. Provide it
  with paths to transcript brief, web research brief, Obsidian note content,
  style guide, outline template, and memory files. Before marking the task
  complete, the writer self-reviews: verify all user decisions from the spawn
  prompt are applied (terminology, scope, weight) and cross-reference concrete
  numbers against the web research brief.

**Check-in with Vincent:** Present the first draft outline and ask if the
direction is right before entering the critique-revise loop.

#### Phase 3: Critique-revise loop (up to 3 rounds)

Spawn the **style critic**. Each round, create all tasks upfront with
dependencies (`addBlockedBy`) to enforce ordering:

- **Task: Verify claims (round N)** → web researcher (if claims were
  flagged). Updates the research brief.
- **Task: Revise outline (round N)** → outline writer. Include revision
  instructions from Vincent's feedback (round 1) or the previous critic
  verdict. `addBlockedBy` verify claims task if it exists.
- **Task: Review outline (round N)** → style critic. `addBlockedBy` revise
  task. The critic produces:
  1. **Style feedback** → messages the outline writer
  2. **Claims list** → messages the web researcher
  3. **Verdict** → messages the skill leader

The loop exits when the critic says "ready to present" or "present next round",
or after 3 rounds (whichever comes first).

#### Phase 4: Present to user

Deliver the final outline with a summary of:

- What the critic flagged across rounds
- What was addressed and what remains open
- Any unverified claims or gaps

Shut down all teammates and clean up the team with `TeamDelete`.

### Workflow 2: Writing from Outline

Use when: User has an approved outline in `drafts/outlines/` and wants the full post written.

**Steps:**

1. **Absorb style** — Read 2-3 recent posts (if not already done during outline phase)

2. **Write post** — Follow the outline section by section:
   - Expand main messages into full prose
   - Add transitions between sections
   - Include planned diagrams/tables
   - Match voice and formatting from `.claude/docs/writing-style.md`

3. **User review** — User reads draft and provides feedback or direct edits

4. **Iterate** — Address feedback until user approves

5. **Finalize** — Create final post at `content/blog/slug-matching-title.md`

### Workflow 3: Editing an Existing Post

Use when: User wants targeted changes to a published or draft post.

**Steps:**

1. **Read the post** — Understand current content, structure, and tone

2. **Clarify scope** — If unclear, ask user what specifically needs changing

3. **Make targeted edits** — Change only what's requested; don't restructure or rewrite unless asked

4. **User review** — User approves changes

## After Writing

If Vincent made style corrections or expressed preferences during the session,
append them to the "Learnings" section in `.claude/docs/writing-style.md`.

## Style

See `.claude/docs/writing-style.md` for the full style guide (voice, structure,
formatting, technical content, learnings).

## Frontmatter

```toml
+++
title = "Post Title"
date = YYYY-MM-DD
description = "One sentence hook"

[taxonomies]
tags = ["lowercase-tag"]  # typically 1-2 tags

[extra]
katex = true  # only if using math
+++
```

## Output

- **Outlines** go to `drafts/outlines/topic-name.md`
- **Final posts** go to `content/blog/slug-matching-title.md`

Don't run `zola serve` or `zola check` during editing; Vincent prefers to run these himself. Batch validation at the end if needed.
