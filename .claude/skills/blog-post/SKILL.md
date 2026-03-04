---
name: blog-post
description: Create blog post outlines and drafts for Vincent's Zola blog. Orchestrates agent teams for research, outline creation, and style review. Use when asked to create a new outline, write or edit a blog post, or help with blog content.
---

# Blog Post Skill

Orchestrate blog post creation using agent teams. The primary workflow spins up parallel research agents, synthesizes their output into an outline, and runs a critique-revise loop before presenting to the user.

For long/complex topics, propose splitting into multiple posts before outlining.

## Workflows

### Workflow 1: Creating an Outline

See [workflow-outline.md](workflow-outline.md).

### Workflow 2: Writing from Outline

See [workflow-writing.md](workflow-writing.md).

### Workflow 3: Editing an Existing Post

Use when: User wants targeted changes to a published or draft post.

**Steps:**

1. **Read the post** — Understand current content, structure, and tone
2. **Clarify scope** — If unclear, ask user what specifically needs changing
3. **Make targeted edits** — Change only what's requested; don't restructure or rewrite unless asked
4. **User review** — User approves changes

**Optional: style critic loop.** When Vincent asks for it (e.g., "use the
critic" or "run it through the critic"), spawn the style critic as a
subagent. The leader makes edits and the critic reviews in a back-and-forth
loop (up to 3 rounds) without Vincent in the middle. This is useful when
Vincent gives a batch of changes and wants the leader and critic to work
through them together, presenting the result only when both are satisfied.

## After Writing

If Vincent made style corrections or expressed preferences during the session,
append them to the "Learnings" section in `.claude/docs/writing-style.md`.

## Style

See `.claude/docs/writing-style.md` for voice, structure, formatting,
technical content, and learnings.

## Output

- **Outlines** go to `drafts/outlines/topic-name.md`
- **Drafts** go to `drafts/posts/topic-name.md`
- **Final posts** go to `content/blog/slug-matching-title.md`

Don't run `zola serve` or `zola check` during editing; Vincent prefers to run these himself. Batch validation at the end if needed.
