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
- **Don't start paragraphs with "But"**—rephrase or restructure to integrate the contrast
- **Avoid "Because" openers**—prefer "X, so Y" constructions; use "Because" sparingly for effect only
- **No sarcastic rhetorical openers**—questions like "So what's the framework good for?" read as dismissive; lead with the payoff directly
- **Contractions naturally**—use don't, can't, won't; uncontracted forms sound stiff
- **Concrete verbs for abstract processes**—prefer physical language ("sanded down" over "improved", "bolted on" over "added", "stripped back" over "simplified")
- **Concrete over abstract**—show a formula or example instead of describing a property in hand-wavy words (e.g., "all-or-nothing operation")
- **Avoid meta-label sentences**—"The useful property:", "What this gives us:", "The framing that makes this click:". Rewrite as direct claims.
- **Vary verbs for cryptographic constraints.** "Force" is overused in crypto writing. Mix in: bind, push, lock, tie, weld, pin, gate, couple. Each has slightly different connotations—"bind" for KoE-style structural binding, "push" for what a check does to the prover, "lock" for something the prover can't take apart.
- **"Cliffhanger" isn't the voice**—Part 1 ended with an open problem, not a cliffhanger. Write the handoff in the same register as the rest of the post.

## Structure

- **Hook**: Open with what sparked the exploration (a tweet, podcast, conversation, problem); promise a concrete reward ("by the end you'll understand X")
- **Motivate each section**: Answer "why am I reading this?" before diving in
- **Don't assume motivation**—if the post says "we move evaluation into curve space," the reader has to know *why* first. Motivation before mechanism, always.
- **Goal-first, mechanism-second, algebra-at-end.** For sections introducing a cryptographic check: (1) state what the honest object should be / what check we want to enable, (2) introduce the tool abstractly, (3) apply to the specific construction, (4) derive consequences algebraically at the end. Don't assert the conclusion in the opener—readers take unjustified claims on faith or mentally flag them; better to have them watch the derivation land.
- **Short sections**: 2-4 paragraphs per `##` section; use questions as section transitions ("But can X do Y?")
- **Announce structure upfront**: When introducing multi-part concepts, state the count and breakdown
- **Don't preview a scaffolding upfront**—avoid "every Greek letter we introduce below will...", "first half / second half". Let the structure emerge from the content as the reader encounters each piece.
- **Examples before definitions**: Build intuition first, then formalize; demonstrate value through a concrete scenario rather than explaining what something is good for
- **Build up to key equations**: show the derivation step by step, then present the clean formula as the payoff; don't drop an equation first and justify it after
- **Don't front-load details**: if a concept (finite fields, specific algorithms) only matters in a later section, let that section introduce it; transitions should do one job: bridge the reader forward
- **Don't introduce a concept before the reader can place it**—mentioning "pairing check" before pairings are set up, or naming specific SNARK variants (Pinocchio, BCTV14) in the intro before the reader knows what those are, makes the reader stall.
- **Numbers need context**: don't quote concrete numbers (byte sizes, gas costs) before the reader has the machinery to understand where they come from; stay conceptual in setup sections
- **Late sections shed complexity**: the reader is fatigued after hard technical content; closing/future sections should use plain language, avoid new jargon, and strip optional details
- **Specific section titles**: Capture the actual content; generic titles like "On This Blog" or "The Full Loop" are weaker than descriptive ones. Narrative titles ("The Trusted Setup Has a Secret") read dorky next to ones that name the move ("Moving Evaluation into Curve Space").
- **Parallel section titles for parallel mechanisms.** When sections apply different constraints to the same object (e.g., "Binding $\pi_A, \pi_B, \pi_C$ to the QAP Polynomials" and "Binding $\pi_A, \pi_B, \pi_C$ to a Single Witness"), use the same grammatical structure. Reader instantly sees the second constraint as a sibling to the first, not a new concept.
- **Parallel structure**: When covering related topics in one section, use parallel framing ("On X:... On Y:...") to tie them together
- **Don't restate the same claim across a section boundary.** Section N's closer and Section N+1's opener shouldn't say the same thing in different words. The handoff should add new info (e.g., attack consequence, next question), not rehash.
- **Footnotes for asides**: Keep tangents out of the main flow
- **Anticipate reader "why?" questions.** Choices that might prompt a reader to stop and ask "why?" (why only $\pi_A$ gets the public/private split? why three independent δ's? why this check and not another?) deserve explicit answers—usually a footnote, occasionally one line in prose. Readers who don't ask skip; readers who do get a crisp answer instead of stalling on the construction.
- **Flag simplifications in footnotes; defer rigor to the appendix.** When the body's derivation uses an assumption that isn't strictly true in all cases (e.g., linear independence of per-variable polynomials), acknowledge it in a footnote and point to the appendix for the full story. Don't hide simplifications silently.
- **Media continuity**: When embedding audio/video, keep follow-up commentary attached to preceding context; don't orphan explanatory sentences after media blocks
- **Natural transitions**—no mechanical connectors ("Furthermore", "Additionally", "Moreover"); let the logical flow carry the reader forward
- **Closing section**: "Takeaway", "Bottom Line", or "What's Next" (for series); avoid punchy slogans ("prove more, store less")—plain language beats sound bites after dense technical content
- **When noting a directional shift, state why**: "X is unlikely" is less satisfying than "X is unlikely because Y"
- **Don't reference terms before context**: even casual mentions (e.g., "trie" in an intro) can confuse if unexplained
- **Precision matters in teasers**—a teaser that enumerates gaps/problems must enumerate them accurately. Conflating two different issues (or skipping one) damages the reader's map of what's coming. Enumerate accurately or don't enumerate.
- **Footer**: Auto-generated by the site template. Do not add manually.

## Technical Content

- Explain domain-specific notation piece by piece; use consistent notation throughout
- Use concrete examples and anthropomorphizations
- Explain concepts before naming them; show the technique first, then give it a name. Use plain language before introducing terminology
- **Introduce terminology through action**: "Alice wants to **open** the commitment" beats "In cryptography, revealing a value is called **opening**"; show the term in use
- **Introduce new notation when it clarifies semantic roles.** When a prover's free choice interacts with already-fixed witnesses, give them different letters (e.g., $r_i$ for the bundle weight, $s_i, s_i', s_i''$ for the existing commitment weights). Makes the "four things collapse to one" visible rather than implicit.
- **Unfamiliar terms in late sections**: frame as variants or swaps, not prerequisites ("swaps in different building blocks: a different X, Y, Z")
- Define terms that seem obvious but aren't; geometric or informal language may need algebraic clarification
- **Define cryptographic symbols where first mentioned, not later.** If $G_2$ first appears in §2's pairing check, the reader gets derailed. Introduce it in §1 where $\mathbb{G}_2$ is named, so the symbol is earned before use.
- List edge cases explicitly; don't leave them implicit
- When contrasting concepts, explain WHY the distinction matters
- Don't make unsubstantiated claims; if something hasn't been proven in the post, don't assert it
- **Never present incomplete machinery as a working protocol**: if the post hasn't built enough to make something work, say so upfront rather than presenting it as working and then poking holes
- **Complexity claims need honesty**: if an operation is O(n), don't call it constant; be precise about what each stage achieves and what's deferred
- Qualify claims about real-world applications; distinguish the mathematical foundation from implementation details
- **Production numbers ground abstract claims**: when arguing succinctness or scale, include real-world circuit sizes (e.g., Zcash ~1.5M gates, zkEVM rollups tens of millions)
- Acknowledge hard problems honestly in vision/future sections; shows intellectual honesty without undermining the argument
- Proofs: rigorous but followable; use "Suppose, toward contradiction" phrasing
- **Honest attack descriptions over hand-waving.** When explaining a cryptographic flaw or fix, describe the concrete attack path (what the prover manipulates, why the math lets them, what the fix prevents). "A second secret γ blocks this" is weaker than "γ introduces factors the prover doesn't know, so their dependency-exploiting shift can't be reconstructed."
- **Ground security guarantees in concrete attacks.** When motivating a property (soundness, zero-knowledge, input pinning), state the attacks the property defeats rather than the abstract guarantee. "An attacker with a candidate witness can confirm it by recomputing $\pi_A$" lands harder than "the commitments leak information." The concrete attack makes the next section feel necessary rather than academic.
- **Show the algebra when it's compact.** Soundness/privacy arguments often read handwavy ("divisibility survives because $Z$ vanishes at gates"). When the algebra fits in one or two display equations, show it—the factoring $(L + \delta_L Z)(R + \delta_R Z) - (O + \delta_O Z) = Z \cdot (H + \ldots)$ is both tighter and more convincing than prose about why it factors.
- Link to related posts with explicit names: `[my post on Russell's Paradox](@/blog/russells-paradox.md)`
- **Link on the concept, not the container**: `[finite field](@/blog/...)` beats `my post on [X](@/blog/...)` or `[post](@/blog/...)`; the link text should be the most informative word
- **Semantic pointers beat numbered section references.** Avoid "§N" or "section N" in prose—they don't render as links, and readers have to count sections to find them. Use named anchors ("the divisibility check," "the β-derivation") that carry meaning on their own and survive section renumbering.
- **Bridge, don't re-explain**: when a concept was covered in an earlier post, link to it with a one-sentence reminder and move on; keeps new posts shorter and rewards returning readers. This extends across posts in a series: Part 2's reader has Part 1 loaded. Re-bolding a term that Part 1 already bolded is a "first use" signal that contradicts continuity.
- **Don't repeat formulas**: if an expression just appeared, use "this $C$" or prose instead of restating it
- **Direct assignment for value changes, not "in place of."** When a commitment's definition changes ($\pi_H$ was $H(\tau) G_1$, becomes $H_\text{new}(\tau) G_1$), write "the prover now computes $\pi_H = H_\text{new}(\tau) G_1$," not "commits to $H_\text{new}(\tau) G_1$ in place of $\pi_H$." "In place of" reads as a different element replacing an existing one, not as a redefinition.
- **Repeat-check within short paragraphs and bullet lists**—if the same term (L/R/O, "aggregate", "evaluation") appears more than once in a tight span, refactor. The reader tracks the repetition and infers emphasis where none is intended.
- **Don't invent pedagogical metaphors outside the literature's own vocabulary**—if a concept has a standard name (e.g., "Knowledge of Exponent," "α-shifted pair," "knowledge commitment"), use it instead of a narrative analogy ("watermark"). Analogies that aren't how practitioners think confuse cross-reference and feel forced.
- **Literature notation is optional when used once.** If a standard-literature symbol ($\mathbf{vk}_{\text{IC},i}$, $A_\text{mid}$, etc.) appears only once or twice in the body and isn't reused, use plain notation in the body and park the literature name in a footnote. The bridge for researchers belongs in the footnote; the main text shouldn't carry terminology that doesn't pull its weight.
- **If a running example isn't actively teaching in a section, drop the thread there**—callbacks like "Part 1's circuit doesn't have this" or "t = 1, 2, 3, 4 for Part 1's circuit" cost reader attention without earning their keep. Abstract sections can stay abstract.
- **For multi-part posts, the connection to earlier parts should be structural** (same machinery, same variables, same identities), not numeric. The intro callback does the connective work; grafting numeric callbacks into later sections dilutes both.
- **Consistency of generator notation across a series.** If an earlier post in the series uses $G_1, G_2$, stick with it. Don't mix in $\mathcal{P}_2$ or other conventions without earning them.
- **Frame superseding tech as evolution**: when mentioning technology that supersedes what the post teaches, frame the current content as foundational, not obsolete; don't undercut the reader's investment

## Math-Heavy Posts

- Use display math liberally; equations should be easy to spot, not buried in prose
- Use bullet points for lists of examples, axioms, verification steps; less prose for technical content
- **Prefer $\forall i$ over $\sum_i$ when reasoning term-by-term.** If the argument works per-term in a sum (coefficient matching, per-variable identity), state the per-term identity with "$\forall i$"—more direct than asserting equality of sums and then unpacking.
- **Use ≡ (\equiv) for algebraic identities, = for defined equalities.** When showing that a constructed object is equivalent to a compact form by algebraic manipulation, ≡ signals "these are the same thing computed two ways." Reserves = for definitions and derivations where equality is the conclusion, not an identity.
- **Label compound expressions with underbraces.** When a display equation introduces a polynomial or aggregate that the following paragraph will refer to by name (e.g., $L_\text{new}, R_\text{new}, O_\text{new}, H_\text{new}$), label it in-place with `\underbrace{...}\_{name}` so the follow-up prose can use the short name without restating the full expansion. Deloads repetition and gives the reader's eye a named anchor to point back to.

## Formatting

- **Bold** key terms on first use
- Avoid em dashes; use colons, semicolons, periods, or parentheses instead. On the rare occasion one is needed, use `—` (em dash character), never `--` (double hyphen)
- Use `\tag{N}` for important equations referenced later; refer back with "equation $(N)$", not bare "$(N)$"
- **Consistent terminology**: pick one term for a concept (e.g., "public parameters" not sometimes "public points"); bold on first use
- KaTeX: `$...$` inline, `$$...$$` block; use `\lbrace`/`\rbrace` for set braces, `\*` for Kleene star
- KaTeX underscore escaping: inside `\text{}`, use `\text{node\\_hash}`; for a subscript that follows any closing `}` (e.g., `\text{child}_0`, `\underbrace{...}_{\text{public}}`, `\mathbf{vk}_x`), escape the underscore as `\_` — pulldown-cmark's emphasis pairer scans `}_..._` patterns across math and can scramble the line before KaTeX runs, especially when several such subscripts appear in the same block
- KaTeX array line breaks: use `\\\` (three backslashes) inside `\begin{array}` environments; prefer `\begin{array}{ll}` with `\left\lbrace...\right.` over `\begin{cases}` which is flaky
- **Zola / pulldown-cmark quirk for KaTeX aligned line breaks.** Inside `\begin{aligned}`, use `\\\\` (four backslashes) to get a single `\\` through the markdown processor. Attempting row spacing with `\\[0.5em]` fails because the bracketed param gets eaten as a markdown reference. Use `\\\\[0.5em]` instead if spacing is needed.
- **Mobile rendering constraint: aligned environments must have narrow rows.** Wide equations overflow on iPhone. Split long sums with `aligned`, or drop redundant terms (e.g., don't show the expanded form and the compact form side-by-side when the prose already explained the reduction).
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

<!-- New learnings from ongoing sessions go here. Consolidated into the Style
sections above at the start of each new outline session. -->

- **Sequential-post intros can skip external hooks.** For series posts, Part N can open with a structural bridge from Part 1 (state the handoff identity, name what remains) without a fresh tweet/podcast/anecdote. The reader has already opted in; terseness is fine.
- **Inline gloss over parenthetical bracket.** When defining a term, weave the definition into sentence syntax. "Working in two prime-order elliptic-curve groups $\mathbb{G}_1$ and $\mathbb{G}_2$, the setup..." beats the same gloss stuffed in parentheses. Brackets interrupt; inline integrates.
- **Appendix preambles earn one sentence.** One line naming what's in each appendix beats multi-paragraph framing. Readers scanning appendices want an index, not prose.
- **Load-bearing test before including an appendix.** If an appendix requires vocabulary the body doesn't cover (q-PDH, δ-domain-separation, reduction-based security) and the concept isn't reused, prefer a one-line footnote with an external pointer. Don't drag in foreign machinery for a single side-note.
- **Verify quote attributions match your scope.** When paraphrasing a paper's phrasing, confirm the quote is actually about your version of the concept, not a structurally similar but scope-different one. (Groth16's γ quote is about γ+δ jointly; citing it as γ-only misleads.)
- **Example should match the post or match current reality.** A real-world reference should either (a) run the exact machinery the post taught, or (b) be what's currently deployed. A historical intermediate that's neither (e.g., Zcash Sapling for a BCTV14 post — Sapling was Groth16, and current Zcash is Orchard) confuses the reader about what they just learned. When neither (a) nor (b) fits cleanly, drop the named version and use a generic descriptor ("a private-payment circuit") instead.
- **When verifying citations, account for the ePrint submission-year vs publication-year gap.** ePrint URLs (`/2012/215`) use submission year; the paper's conventional citation uses publication year ("GGPR 2013"). Both are correct for the same paper; write it as "GGPR §5.3 (EUROCRYPT 2013; ePrint 2012/215)" so the mismatch is self-explanatory.
