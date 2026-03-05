# Writing Style Guide

This is the canonical reference for Vincent's blog writing style. Used by the
blog-post skill and the style-critic agent.

## Updating This File

If Vincent makes style corrections or expresses preferences during a session,
append them to the "Learnings" section at the bottom. Keep entries concise
(1-2 lines each). All learnings are valid immediately.

**Consolidation** happens separately, when the style-critic agent runs at the
start of a new outline session. It reads both sections and:

- Moves Learnings entries into the appropriate Style subsections (Voice,
  Structure, Formatting, etc.)
- Combines related entries and removes duplicates with existing Style rules
- Only removes a Learning if Vincent has explicitly contradicted it with a
  newer preference

## Voice

- **Conversational but precise**—explaining to a curious friend, not lecturing
- **First person**—"I want to explore", "Let's see what this means"
- **Humble curiosity**—share the learning journey, not just conclusions
- **Dense, no fluff**—respect the reader's time; don't say the same thing twice in different words; don't state the same fact in multiple sections (consolidate to where it has the most impact)
- **Precise surprise**—before writing "how can X do Y?", ask: does something the reader already knows also do Y? If so, sharpen the claim to what's genuinely new
- **Honest about difficulty**—don't claim something is "expected" or "obvious" when it's actually surprising
- **Flowing rhythm**—prefer flowing constructions ("Not exactly breaking, more like resurfacing") over terse ones; answer your own rhetorical questions to drive forward rather than leaving them hanging
- **Concrete framing**—ground descriptions in what's happening ("The flow looks like this"), not how you feel about it ("The vision I keep returning to")
- **One critique, then constructive**—when critiquing external work, one jab is enough. Frame as "opens a door" not "failed to deliver"
- **Genuine acknowledgment**—when pivoting from criticism to opportunity, be honest about the limitation first
- **Soften alignment claims**—"It looks like we're circling similar ideas" beats asserting alignment
- **Don't double-hedge**—one hedge per claim is enough
- **Sentence fragments sparingly**—starting with "And" can work for effect, but when in doubt, integrate into the previous sentence
- **Avoid "Because" openers**—prefer "X, so Y" constructions; use "Because" sparingly for effect only
- **No sarcastic rhetorical openers**—questions like "So what's the framework good for?" read as dismissive; lead with the payoff directly
- **Contractions naturally**—use don't, can't, won't; uncontracted forms sound stiff
- **Concrete verbs for abstract processes**—prefer physical language ("sanded down" over "improved", "bolted on" over "added", "stripped back" over "simplified")
- **Concrete over abstract**—show a formula or example instead of describing a property in hand-wavy words (e.g., "all-or-nothing operation")

## Structure

- **Hook**: Open with what sparked the exploration (a tweet, podcast, conversation, problem); promise a concrete reward ("by the end you'll understand X")
- **Motivate each section**: Answer "why am I reading this?" before diving in
- **Short sections**: 2-4 paragraphs per `##` section; use questions as section transitions ("But can X do Y?")
- **Announce structure upfront**: When introducing multi-part concepts, state the count and breakdown
- **Examples before definitions**: Build intuition first, then formalize; demonstrate value through a concrete scenario rather than explaining what something is good for
- **Build up to key equations**: show the derivation step by step, then present the clean formula as the payoff; don't drop an equation first and justify it after
- **Don't front-load details**: if a concept (finite fields, specific algorithms) only matters in a later section, let that section introduce it; transitions should do one job: bridge the reader forward
- **Numbers need context**: don't quote concrete numbers (byte sizes, gas costs) before the reader has the machinery to understand where they come from; stay conceptual in setup sections
- **Late sections shed complexity**: the reader is fatigued after hard technical content; closing/future sections should use plain language, avoid new jargon, and strip optional details
- **Specific section titles**: Capture the actual content; generic titles like "On This Blog" or "The Full Loop" are weaker than descriptive ones
- **Parallel structure**: When covering related topics in one section, use parallel framing ("On X:... On Y:...") to tie them together
- **Footnotes for asides**: Keep tangents out of the main flow
- **Media continuity**: When embedding audio/video, keep follow-up commentary attached to preceding context; don't orphan explanatory sentences after media blocks
- **Natural transitions**—no mechanical connectors ("Furthermore", "Additionally", "Moreover"); let the logical flow carry the reader forward
- **Closing section**: "Takeaway", "Bottom Line", or "What's Next" (for series); avoid punchy slogans ("prove more, store less") — plain language beats sound bites after dense technical content
- **When noting a directional shift, state why**: "X is unlikely" is less satisfying than "X is unlikely because Y"
- **Don't reference terms before context**: even casual mentions (e.g., "trie" in an intro) can confuse if unexplained
- **Footer**: Auto-generated by the site template. Do not add manually.

## Technical Content

- Explain domain-specific notation piece by piece; use consistent notation throughout
- Use concrete examples and anthropomorphizations
- Explain concepts before naming them; show the technique first, then give it a name. Use plain language before introducing terminology
- **Introduce terminology through action**: "Alice wants to **open** the commitment" beats "In cryptography, revealing a value is called **opening**"; show the term in use
- **Unfamiliar terms in late sections**: frame as variants or swaps, not prerequisites ("swaps in different building blocks: a different X, Y, Z")
- Define terms that seem obvious but aren't; geometric or informal language may need algebraic clarification
- List edge cases explicitly; don't leave them implicit
- When contrasting concepts, explain WHY the distinction matters
- Don't make unsubstantiated claims; if something hasn't been proven in the post, don't assert it
- Qualify claims about real-world applications; distinguish the mathematical foundation from implementation details
- Acknowledge hard problems honestly in vision/future sections; shows intellectual honesty without undermining the argument
- Proofs: rigorous but followable; use "Suppose, toward contradiction" phrasing
- Link to related posts with explicit names: `[my post on Russell's Paradox](@/blog/russells-paradox.md)`
- **Link on the concept, not the container**: `[finite field](@/blog/...)` beats `my post on [X](@/blog/...)` or `[post](@/blog/...)`; the link text should be the most informative word
- **Bridge, don't re-explain**: when a concept was covered in an earlier post, link to it with a one-sentence reminder and move on; keeps new posts shorter and rewards returning readers
- **Don't repeat formulas**: if an expression just appeared, use "this $C$" or prose instead of restating it
- **Frame superseding tech as evolution**: when mentioning technology that supersedes what the post teaches, frame the current content as foundational, not obsolete; don't undercut the reader's investment

## Math-Heavy Posts

- Use display math liberally; equations should be easy to spot, not buried in prose
- Use bullet points for lists of examples, axioms, verification steps; less prose for technical content

## Formatting

- **Bold** key terms on first use
- Avoid em dashes; use colons, semicolons, periods, or parentheses instead. On the rare occasion one is needed, use `—` (em dash character), never `--` (double hyphen)
- Use `\tag{N}` for important equations referenced later; refer back with "equation $(N)$", not bare "$(N)$"
- **Consistent terminology**: pick one term for a concept (e.g., "public parameters" not sometimes "public points"); bold on first use
- KaTeX: `$...$` inline, `$$...$$` block; use `\lbrace`/`\rbrace` for set braces, `\*` for Kleene star
- KaTeX underscore escaping: inside `\text{}`, use `\text{node\\_hash}`; for subscripts after `\text{}`, use `\text{child}\_{0}` not `\text{child}_0`
- KaTeX array line breaks: use `\\\` (three backslashes) inside `\begin{array}` environments; prefer `\begin{array}{ll}` with `\left\lbrace...\right.` over `\begin{cases}` which is flaky
- Escape tildes for approximation: use `\~` instead of `~` (e.g., `\~100 GB`) to avoid strikethrough interpretation
- Numbered lists that span non-list content: use HTML `<ol start="N">` to continue numbering
- `<details><summary>...</summary>...</details>` for optional deep-dives
- Tables: include a line explaining how to read them
- Twitter/X embeds: use `data-theme="dark"` and `data-align="center"`; needs CSP config for `platform.twitter.com`
- Anchor links: use standalone `<a id="..."></a>` elements; `id` on other elements doesn't work reliably in Zola
- No emojis

## Series Posts

- Title format: "Title (Part N/M)"
- Link to previous/next parts at top and bottom
- Each part should stand alone while building on prior context

## Banned Phrases

Never use these. They are recognizable LLM-isms or empty filler.

### Dead AI language

"delve", "dive into", "unpack", "harness", "leverage", "utilize", "landscape",
"realm", "robust", "game-changer", "cutting-edge", "straightforward",
"it's important to note", "it's worth noting", "in order to",
"in today's [anything]"

### Dead transitions

"Furthermore", "Additionally", "Moreover", "Moving forward",
"At the end of the day", "To put this in perspective",
"What makes this particularly interesting is", "The implications here are",
"In other words", "It goes without saying"

### Engagement bait

"Let that sink in", "Read that again", "Full stop",
"This changes everything", "Are you paying attention?"

### AI cringe

"Supercharge", "Unlock", "Future-proof", "10x your productivity",
"The AI revolution", "In the age of AI"

### Generic insider claims

"Here's the part nobody's talking about", "What nobody tells you",
anything with "nobody" or "most people don't realize"

### Negate-and-reframe (fatal)

"This isn't X. This is Y." and all variants: "Not X. Y.", "Forget X. This is Y.",
"Less X, more Y." Never negate one framing to assert a corrected one. Just state
the positive claim directly.

---

## Learnings

- **No em dashes.** Use periods, commas, or semicolons instead.
- **Never present incomplete machinery as a working protocol.** If the post hasn't built enough to make something work, say so upfront rather than presenting it as working and then poking holes.
- **Don't start paragraphs with "But".** Rephrase or restructure.
- **Complexity claims need honesty.** If an operation is O(n), don't call it constant. Be precise about what each stage achieves and what's deferred.
- **Production numbers ground abstract claims.** When arguing succinctness or scale, include real-world circuit sizes (e.g., Zcash ~1.5M gates, zkEVM rollups tens of millions).
