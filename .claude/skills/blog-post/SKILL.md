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
| **Transcript researcher** | `transcript-researcher` agent | Parse a conversation transcript to understand what Vincent found most interesting, what he asked about repeatedly, and what confused him. Produce a prioritized brief telling the outline writer what to cover and in what order. Remain on standby to answer questions from other agents about Vincent's interests. | `drafts/briefs/<topic>-transcript-brief.md` |
| **Web researcher** | Ad-hoc `general-purpose` agent | Ensure technical correctness and completeness of the outline. In phase 1: build foundational understanding of the topic through authoritative sources. In phase 3: verify specific claims, fill gaps, and check completeness issues flagged by the style critic. | `drafts/briefs/<topic>-web-research.md` (updated across phases) + `SendMessage` to outline writer when brief is updated |
| **Outline writer** | Ad-hoc `general-purpose` agent | Synthesize all research into a draft outline, then revise based on critique. | `drafts/outlines/<topic>.md` |
| **Style critic** | `style-critic` agent | Review the outline for style alignment and pedagogical flow. Flag technical claims that need verification (for the web researcher). Produce a verdict on readiness. | Feedback via `SendMessage` |

#### Consulting Vincent

Vincent wants to be consulted during outline creation. Two mechanisms:

- **Phase boundary check-ins** — The skill leader checks in with Vincent at
  the end of phase 1 (research summary) and phase 2 (first draft). Present
  what was found/written, flag open questions, and ask if the direction is
  right before proceeding.
- **Blocker escalation** — If any agent hits a blocker mid-phase (ambiguous
  scope, contradictory sources, unclear priorities), it messages the skill
  leader, who relays the question to Vincent via `AskUserQuestion`.

#### Phase 1: Research (parallel)

Create a team with `TeamCreate`, then run these in parallel:

1. **Transcript researcher** — Give it the high-level topic and the transcript
   path. It parses the conversation and writes a prioritized brief: what the
   outline should cover, what Vincent was especially engaged with, and what
   angles to consider. Stays on standby after delivering the brief. **Skip
   entirely if no transcript was provided.**

2. **Web researcher** — Give it the topic, any user-provided references, and
   a summary of what the post will cover. It finds authoritative sources
   (papers, specs, prior art, related blog posts) to ground the outline's
   technical claims. Writes an annotated research brief.

3. **Obsidian lookup** (skill leader) — Search for relevant notes in Vincent's
   Obsidian vault `Study/` folder using the Obsidian skill. Pass any matching
   note content to the outline writer in phase 2.

**Check-in with Vincent:** Summarize what the researchers found, what the
Obsidian notes contain (if any), and flag any open questions (e.g., ambiguous
scope, conflicting sources, topics that could go multiple directions). Wait
for Vincent's input before proceeding.

#### Phase 2: Outline writing

Spawn the **outline writer**. Provide it with:

- Transcript brief from `drafts/briefs/` (if any)
- Web research brief from `drafts/briefs/`
- Obsidian note content (if any)
- Style guide at `.claude/docs/writing-style.md`
- Outline template at `.claude/skills/blog-post/outline-template.md`
- Memory files for series context (notation, scope boundaries from prior sessions)

The writer synthesizes all inputs into a first draft outline at
`drafts/outlines/<topic>.md`.

**Check-in with Vincent:** Present the first draft outline. Ask if the
structure, scope, and emphasis are on the right track before entering the
critique-revise loop.

#### Phase 3: Critique-revise loop (up to 3 rounds)

Spawn the **style critic**. Each round:

1. **Style critic** reviews the outline and produces:
   - **Style feedback** to the outline writer: prioritized as must fix /
     should fix / consider
   - **Claims list** to the web researcher: technical claims to verify, gaps
     to fill, completeness issues to check
   - **Verdict**: one of:
     - *Ready to present* — outline is good enough for Vincent
     - *Present next round* — needs minor work but worth checking with Vincent
       after one more pass
     - *Needs another round* — significant issues require further revision

2. **Web researcher** verifies claims from the critic's list and updates
   `drafts/briefs/<topic>-web-research.md`. Then sends a `SendMessage` to the
   outline writer summarizing what changed and asking it to re-read the brief.

3. **Outline writer** re-reads the updated web research brief, then revises
   based on style feedback and the new findings.

The loop exits when the critic says "ready to present" or "present next round",
or after 3 rounds (whichever comes first).

#### Phase 4: Present to user

Deliver the final outline with a summary of:

- What the critic flagged across rounds
- What was addressed and what remains open
- Any unverified claims or gaps

Shut down all teammates.

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
