---
name: blog-post
description: Create blog post outlines and drafts for Vincent's Zola blog. Orchestrates agent teams for research, outline creation, and style review. Use when asked to create a new outline, write or edit a blog post, or help with blog content.
---

# Blog Post Skill

Orchestrate blog post creation and editing. Three workflows cover the full
lifecycle: outlining (research + critique), writing from an approved outline,
and editing existing posts. Each workflow is detailed in its own file.

## Workflows

### Workflow 1: Creating an Outline

See [workflow-outline.md](workflow-outline.md).

### Workflow 2: Writing from Outline

See [workflow-writing.md](workflow-writing.md).

### Workflow 3: Editing an Existing Post

See [workflow-editing.md](workflow-editing.md).

## Team Retrospective

Before shutting down a team, run a brief retro:

1. Ask each teammate what they'd improve about the workflow.
2. Ask Vincent if he has feedback on how the process went.
3. Append all suggestions to `.claude/docs/workflow-retros.md`.

This applies to any workflow that uses a full team (outline, writing). Skip
for lightweight flows like editing without the critic loop.

## After Each Session

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
