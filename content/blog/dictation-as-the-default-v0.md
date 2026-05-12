+++
title = "Dictation as the Default v0"
date = 2026-05-05
draft = true
<!-- FEEDBACK: The description is too long. Look for a previous blog post description to come up with something that is more consistent.  -->
description = "How I came to dictate most of what I write — across three languages, into Claude Code, into messages — and why a thin BYOK client over swappable providers is what made the setup keep getting better."

[taxonomies]
tags = ["ai", "dev"]
+++

<!-- FEEDBACK: The tweet I posted last summer isn't in the center testimonial on their page anymore; they changed it a few days ago. It's still somewhere, but not displayed as prominently. I would rather not have that in the first paragraph—let's move it to an Easter egg or a subdued disclaimer that this is the only tie I have with the site.  -->
I've been dictating most of what I write through VoiceInk, an indie macOS dictation tool — and a tweet I posted last summer ended up as the center testimonial on their homepage. I just saw a few weeks ago that I had been picked up by the developer. It was very funny. I have no relationship with him beyond being a fan of his work; getting that out of the way before the rest of the post.

What's actually kept me on this setup is structural: VoiceInk is a thin client over providers I bring my own keys to, so the moment a better transcription or enhancement model ships, I can point at it the same afternoon. That's the argument of the post. The cost numbers are the receipt later.

By the end you'll know exactly what I run, why each layer is swappable, what the whole rig has cost me over three years, and where it still falls over.

Siri's latency and the way it kept botching word boundaries had been bothering me for years — having worked on ASR at Amazon, I knew we should be in a better place than that.

## The setup, top to bottom

I'm dictating this post through the stack I'm about to describe. When I press my shortcut, audio goes from a Sennheiser Profile USB into VoiceInk, which hands it off to ElevenLabs Scribe v2 Realtime for transcription, then runs the result through Gemini 3.1 Flash-Lite with a short custom prompt, and pastes the cleaned-up text into whatever app is in front of me. Whisper.cpp is still in there as a local fallback if I'm offline. Four layers, one shortcut.

<!-- FIGURE: stack diagram — four layers (mic → transcription → enhancement → paste target), current choice in each slot. -->

Case in point: the first time I dictated this post's prompt to Claude, VoiceInk rendered its own brand phonetically as "Voice Inc." I need to add that to the app's dictionary.

VoiceInk is a one-developer show by Prakash Joshi Pax, and it doesn't do much marketing — I landed on the GitHub repo through the usual mix of searching and asking Claude. The source is GPL v3, so you can clone the repo and build it yourself for free; the paid license ($25 for one Mac, up to $49 for the multi-Mac tier) pays the developer for updates, support, and the time to keep working on the project. I'd watched a few of his demo videos and I wanted to encourage him, and $25 was not a large price to pay to give this a try.

One more thing about the setup: VoiceInk is BYOK. I bring my own API keys for transcription and enhancement, and the bills land on my own accounts.

Three of those four layers got dramatically better in the last twelve months. Worth saying which ones, and why I noticed.

## Why now?

- models: I started with Ollama running a quantized Whisper locally. It wasn't fast enough on my Mac, so I graduated to using Groq for the same model, unquantized. That worked well until recently, when the latency crept up and the quality dropped enough that I started reaching for it less. I tried a handful of other API services, and the one that stuck is Scribe v2 from ElevenLabs. ElevenLabs shipped Scribe v2 Realtime on January 6th of this year (verify this).

- Real-time models, like the one I'm using right now, are making a huge difference because my voice is passed to the model as I'm speaking. Whereas before, it would wait until I finished speaking and then send the audio file through the wire for transcription. By the time I'm done speaking, the large majority of my spoken tokens are already transcribed, which gives a huge gain in speed. The average transcription I'm getting for a couple of sentences from that model is about 300 milliseconds per utterances.

- enhancements: Explain how enhancement works. I mentioned that the Gemini 3.1 Flash Lite is the sweet spot for the accuracy and latency trade-off.  

<!-- FIGURE: two-bar timeline — file-based ASR (segments back-to-back) vs streaming ASR (segments overlapped, ending almost flush with "you stop talking"). -->

you case use some of this for the real-time models part, it's pretty good.
When I'm done speaking, the transcription has already finished its first pass with ElevenLabs. That sentence sounds like a small thing, and it isn't. With file-based ASR, the floor on end-to-end latency is `time-to-finish-speaking + inference time` — the model can't start until the audio file is closed, so the two intervals run back-to-back. Streaming overlaps them. The audio is being transcribed while I'm still talking, and by the time I stop, almost all the work is already behind me. That's a different latency profile.

## Why VoiceInk

- What is VoiceInk: use some of this:
  - VoiceInk is a one-developer show by Prakash Joshi Pax, and it doesn't do much marketing — I landed on the GitHub repo through the usual mix of searching and asking Claude. The source is GPL v3, so you can clone the repo and build it yourself for free; the paid license ($25 for one Mac, up to $49 for the multi-Mac tier) pays the developer for updates, support, and the time to keep working on the project. I'd watched a few of his demo videos and I wanted to encourage him, and $25 was not a large price to pay to give this a try.

- One time fee (add details from Website); you can even build it from source for free (I don't as I like the automatic updates on the app and want to support the developer). I don't see how other companies can justify charging a subscription fee in this space, especially as underlying transcription models improve and get cheaper all the time, all the while giving you less control over it. It doesn't make sense to me.
  - was a big fan of philosophy behind the Once model from DHH <https://world.hey.com/dhh/campfire-is-once-1-d2cebd12>, although Once has since moved away of the one-time fee concept.

- BYOK: Bringing your own key might feel like additional cost and setup work for some users, but it is practically free. The Gemini API calls I'm making cost me less than 10 cents a month, and the ElevenLabs tokens I'm spending come out to around $3 a month or less for someone who uses the service every day.
  - The point of bringing your own key is that it also allows you to swap models for the latest, most performant option whenever it becomes available, which is super relevant these days as different labs are launching new options every day. Case in point, this week, OpenAI's real-time Whisper model was launched. Link to it.

- The developer is super responsive when I have issues. For example, when I had issues with the "pause media while recording" feature, which pauses music apps like Spotify while you're speaking and resumes it when you're done, it was attended to and fixed promptly.

- It also feels good to encourage an indie developer who's not backed by millions of dollars in VC funding. (Wispr Flow has raised $81M to date;). <https://www.prnewswire.com/news-releases/wispr-raises-25m-to-build-its-voice-operating-system-302621858.html>

## The enhancement prompt as a design artifact

Here's the prompt I give the enhancement model:

> - Ignore the part from the system prompt about organizing into short paragraphs. I prefer having everything in one paragraph.
> - Don't make my prose more formal vs what I said. I want you to write casually in English, French, or Japanese.
> - If rambling, my constructions are confusing, convoluted, and there's an opportunity to simplify and streamline, do it. My Japanese is especially wonky and will require more corrections than French or English.
> - If I say at the end, "Fix my rambling," I mean that I want you to actively produce a streamlined version of what you think I intended to say.
> - When enumerating points, use this notation: 1)2)3)

Five bullets, doing more work than they look like they're doing:

- **Subtract the short-paragraph default.** The first bullet doesn't add an instruction, it cancels one VoiceInk ships with. Subtraction is harder than addition because you have to know what defaults you're inheriting before you can turn them off — most prompt advice tells you to be concise or use bullets, almost none tells you to go read the system prompt you're sitting on top of.
- **Casual register across English, French, and Japanese.** Models trained on edited prose drift toward balanced clauses and rounded edges. I want the way I actually talk, in whichever of the three languages I'm in — a casual register I have to ask for explicitly because the gravity pulls the other way.
- **Conditional rewrite when I ramble, more help for Japanese.** If my construction is confusing, simplify it. My Japanese is especially wonky and will require more corrections than French or English, so I name that directly in the prompt rather than pretending the three languages are at parity.
- **"Fix my rambling" as an in-band trigger.** A verbal slash command. Default mode is light cleanup; if I tack "Fix my rambling" onto the end of what I just said, the model switches to actively producing a streamlined version of what it thinks I meant. Two modes, one prompt, decision made while I'm still talking.
- **`1)2)3)` for numbered lists.** Because that's what I actually use when I write.

I've caught the model writing things I didn't say — it's very rare, I catch it immediately, and it's not a big problem.

All of this — model, streaming, enhancement prompt — is something I configured once. The reason I expect it to keep getting better is structural.

## Why the stack stays swappable

In the last few months, I've swapped local Whisper for Groq and then Groq for ElevenLabs at the transcription layer, and I've moved from Gemini 1.5 to 3.1 Flash-Lite at the enhancement layer. None of those moves cost me anything beyond changing a setting. The app didn't need to ship an update. I didn't need to migrate a workflow. The rows in my stack moved; the architecture stayed put.

That's the whole argument. The indie one-time-purchase tool and the BYOK setup aren't two preferences I happen to share — they're the same decision. I want to own the seams between transcription, enhancement, and the client that stitches them together, because the layers underneath are improving fast and on their own schedules. VoiceInk is a thin, well-designed client over whichever ASR and LLM are best this quarter, and Joshi isn't trying to capture the inference layer's margin — that's why the seams stay open. I wanted to encourage him for that, and the $25 felt like a fair way to do it.

When I first looked around, the alternatives — Wispr Flow, Super Whisper — were just subscriptions for a model that I know is open source. That's the part that didn't sit right. A monthly fee bundles the transcription call, the enhancement call, and the app into one number, and once you stop seeing the per-call cost, you stop comparing. The bundle is the product, and the bundle is what makes swapping the model underneath someone else's problem instead of yours.

The receipt, for what it's worth: Gemini runs me about 7–8¢ a month at my volume, Groq's free tier has covered everything so far, and ElevenLabs has been negligible — likely trial credits rather than a permanent free tier, so I'm budgeting for that to change. VoiceInk is $25 once, or zero if you build it from source; it's GPL v3, and the license fee funds the developer rather than gating the software. Compare that with Wispr Flow at $15/month: about $540 over three years, against my roughly $50, most of which went to the mic and the app itself. What a subscription buys that I'm forgoing is real — enterprise compliance, polished cross-platform clients, one bill instead of three accounts to manage — and if any of those matter to you, the math flips.

<!-- FIGURE: swap history table — three rows (transcription, enhancement, hosting/UI), three columns (today, a year ago, likely next). -->

The architecture is what lets the setup keep getting better. The next question is what the setup actually changes about how I work.

## From typing to directing

I write to friends in French, English, and Japanese, and switching keyboard layouts used to be a small slog — my brain having to remember different key positions every time I changed languages. The models VoiceInk runs now auto-detect the language mid-sentence, so the layout switch is gone. I finish speaking and the text appears in whichever language I was in. It feels magical.

The bandwidth of information I can convey per unit of time when I speak is much higher than when I write, so dictation is the more efficient way to work. That's the surface claim, and it's true, but the more interesting thing is what the extra bandwidth actually changes about the work itself.

When typing is the bottleneck, I write a short prompt and iterate. When speaking is cheap, I front-load: the constraints, the edge cases, the tone I want, what I've already tried and why it didn't work, the file paths I expect to be touched. The prompt that used to be three lines is now ten or fifteen, and the model on the other end is the one doing the typing. I'm directing. That's the shift, and it's the part that compounds with every model release — longer, richer instructions pay off more, and speaking is what makes longer instructions feel free.

Concretely: I dictate the prompt and let Claude and Claude Code write 95% of the code and 95% of the prose for my blog posts. The friction is real. There's more back-and-forth editing with Claude than I'd like, and when I get stuck on something small I sometimes patch it by hand instead of arguing with the model. I expect that to keep shrinking as models improve.

(For a sense of the gap: conversational speech runs around 150 words per minute, average typing closer to 40, and Stanford's Ruan et al. 2017 study found dictation roughly 3× faster than typing on mobile even after counting corrections.)

All of which is to say: this works for me. It also fails for me, in specific places, and the failures are sharp enough to be worth naming.

## Where it still fails

Math is the biggest pain point. A lot of my posts include math, and when I want to make an edit and change something significant, I don't have an easy way to dictate `$\rho > \gamma + 1$`. The model has to know I want LaTeX and not prose, that "rho" maps to ρ, that ">" is a symbol and not a word, where the delimiters go. It's an inconvenient limit for someone who writes about cryptography: the more your writing leans on notation, the less dictation buys you.

Small edits in Neovim — fixing a typo, swapping a word, nudging a character — are still faster with the keyboard.

I work from a home office, so I'm not disrupting anyone by talking to my computer all day — that's a privilege, and an open-plan office would make this much harder. Even alone, it felt awkward at first; it's a skill, and it's interesting how you develop the ability to speak without someone responding, while becoming clearer over time.

The mic matters more than people think. I started with a crappy mic embedded in my headphones, and the accuracy was terrible. I moved to the one attached to my camera, and it got better. A couple of days ago I picked up a Sennheiser Profile USB cardioid condenser for around $100, and the difference is clear. If you're trying to make dictation work, upgrade your mic before you upgrade your model.

## Speech and the agentic workflow

Each model upgrade makes this rig pay off more. Longer instructions, with more constraints and more context, reward whatever's on the other end of the prompt — and the better the agent gets, the more there is to gain from feeding it generously. The friction I still hit with small fixes I expect to fade as the models keep improving; what stays is a setup that compounds quietly with every release.

> *[NEEDS VINCENT DICTATION — forward-looking close. Prompt: in two years, do you expect to type at all, except for code character edits and passwords? Is voice becoming the default and keyboard the exception?]*

So I keep dictating, and I keep dictating on purpose. The setup holds because every layer is swappable — the mic, the transcription model, the enhancement model, the agent on the other end of the prompt — and each one keeps getting better on its own schedule. The homepage testimonial I didn't notice for weeks is a fun accident; the architecture underneath is the reason any of the rest of it kept working.
