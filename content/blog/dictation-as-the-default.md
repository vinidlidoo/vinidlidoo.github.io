+++
title = "Dictation as the Default"
date = 2026-05-05
draft = true
description = "How macOS dictation became my default interface, and the swappable BYOK stack that made it stick."

[taxonomies]
tags = ["ai", "dev"]
+++


## Things to add (will remove this section before publishing)

*List of changes, edits, or additions I want to make to the post before publishing*

- I want a post title/description that is a little bit more spicy than this. It should feel like a jab at Wispr Flow and the people using it without naming Wispr Flow.
- Add content about the time save of being able to dictate in auto-detect mode and not having to use a language switcher on the keyboard (and memorize the other languages' keyboard layout as only one can be printed on the physical keys!)
- I want to give an example of using a custom prompt triggered by the word "assistant" that outputs the answer to my question instead of a transcription of what I said. This is a really useful feature.
- I also want to give a single example of a line in my custom prompt that I'm passing to Gemini API for regular transcriptions. Let's use this one: "If my dictation ends with "Fix my rambling," treat everything before it as a rough draft and return a streamlined version of what I meant."
- I want to add the notion that not only is the bandwidth higher when dictating instead of typing, but I will also often dictate content to my computer that I wouldn't type otherwise because of the cognitive load of having to write a couple of sentences for something very simple. It's not only faster, but I also get things done that wouldn't otherwise get done, and that's an important concept. (This is some of the content we had in the V0 file section, from typing to directing.)
- Also, I should mention areas that still need improvement. 1) The ability to write math in LaTeX, which I do commonly on my blog, is not straightforward. 2) A more universal use case would be correctly transcribing words specific to my personal environment, such as people's names or work acronyms. While the custom dictionary feature is helpful, it requires me to add words one by one. I think there's a future where the software proactively asks about words it didn't understand, or provides its best guess and then follows up in a batch at the end of the day or week to build a personalized dictionary.
  - VoiceInk had a feature that would check the word you corrected with your keyboard right after pasting the dictation and update the dictionary automatically, but it was removed recently. I assume it was experimental and not the right solution.

## Intro

I started experimenting with dictation on macOS last summer. Apple's native dictation had been bothering me for a while, and ChatGPT's in-app version made it pretty clear the underlying tech was well ahead of what Apple was shipping. I'd done some ASR benchmarking at Amazon a few years back, which lined up with what I was seeing: state-of-the-art was moving quickly, and consumer apps weren't keeping up.

So I started looking around. Like most people, I evaluated Wispr Flow and the other popular Mac dictation apps before landing on VoiceInk.

For most of the year, I used it sporadically and was happy enough with it. About a month ago, that changed and my usage went up sharply. At this point I'm officially, completely voice-pilled. And I'm glad I picked VoiceInk when I did.

This post walks through my current setup and the choices behind each decision. And my bigger claim: we're at an inflection point where the keyboard is on its way out as the default way we talk to our computers. After forty years of typing, this is one the most consequential shifts I've watched happen in personal computing.

## My Setup

In one sentence: I press the keyboard shortcut, audio goes from a Sennheiser Profile USB into VoiceInk, which hands it off to ElevenLabs Scribe v2 Realtime for transcription, then runs the result through Gemini 3.1 Flash-Lite with a short custom prompt, and pastes the cleaned-up text into whatever app is in front of me. Four layers, one shortcut.

<!-- FIGURE: stack diagram — four layers (mic → transcription → enhancement → paste target), current choice in each slot. -->

From the moment I stop talking, the cleaned-up text lands on screen in one to two seconds, and that latency stays roughly constant whether I dictated a sentence or a full paragraph (more on this in the next section). Accuracy is high enough that I'm correcting maybe a word or two per ten sentences after the Gemini pass.

That combination flipped the keyboard/voice usage ratio for me. I used to write something about 90% using the keyboard and 10% by dictation; today it's essentially the inverse of that. Conversational English runs around [150 to 170 words per minute](https://languagelog.ldc.upenn.edu/myl/ldc/llog/icslp06_final.pdf), while average typing on a desktop keyboard sits closer to [52 wpm](https://userinterfaces.aalto.fi/136Mkeystrokes/resources/chi-18-analysis.pdf). Roughly 3× the bandwidth, which is enough that going back to the keyboard for general prose feels like I'd be insisting on communicating through morse code.

## Why now?

What changed in the past couple of months? Three things got significantly better.

The transcription models have gotten really good. Last summer I started with Ollama running a quantized Whisper locally; nice for privacy, but accuracy wasn't good enough. So I graduated to Groq inference running the same model unquantized. That worked well for a while, until Groq's latency crept up enough that I started reaching for the dictation shortcut less. I then tried a handful of services and settled on [ElevenLabs' Scribe v2 Realtime](https://elevenlabs.io/blog/introducing-scribe-v2-realtime), which shipped late last year.

The big jump with Scribe v2 Realtime was not so much accuracy (it is slightly better) than the streaming architecture I was making use of for the first time. It transcribes the audio while I'm still talking, instead of waiting for the file to close before it starts. With file-based ASR, end-to-end latency is `time-to-finish-speaking + inference time`, the two intervals back-to-back. Streaming overlaps them, so by the time I stop speaking most of the work is already done. In practice I'm seeing about 300 ms per turn for a couple of sentences, enough to feel instantaneous.

<!-- FIGURE: two-bar timeline — file-based ASR (segments back-to-back) vs streaming ASR (segments overlapped, ending almost flush with "you stop talking"). -->

The other part where I made progress is *enhancement*: a second pass VoiceInk runs on the raw transcript to clean up disfluencies, smooth phrasing, and apply a custom prompt (which I'll also get to). The model that landed for me here was Gemini 3.1 Flash-Lite, the cheapest and fastest tier in Google's current lineup. Smart enough to follow a multi-clause prompt and adds about 1 to 1.5 seconds on average, which hits the sweet spot for me on the accuracy/latency curve.

The mic upgrade is the third piece. Modern speech-to-text models are built to hold up under degraded audio (think telephony use cases), but the accuracy drop is never really zero. If you can afford to pay ~$100 to clean up your input, you should (and your Zoom guests will appreciate). A few weeks ago I switched from the mic embedded in my old Sony XM3 headphones to a [Sennheiser Profile USB cardioid condenser](https://amazon.com/dp/B0BTPYCD86), and the output quality improved clearly (I don't like "clearly." Find a better word).

## Why VoiceInk?

As far as I can tell, VoiceInk is a one-developer macOS app by Prakash Joshi Pax. It doesn't do much marketing, and I landed on it the usual way, by searching around and asking Claude. [The source is on GitHub](https://github.com/beingpax/VoiceInk) under GPL v3, so you can clone it and build it yourself for free. The paid license is $25 for one Mac (or $49 for the multi-Mac tier) and goes directly to the developer. The app ships updates every couple of weeks and is already as feature-rich as I need (custom dictionaries, trigger words that route to different custom prompts, [voice activity detection](https://en.wikipedia.org/wiki/Voice_activity_detection), to name a few). I run the paid build because I want those updates yes, but it'd feel wrong not to support the work.

The pricing model is what sold me initially. One-time fee, no subscription, source open if you'd rather not pay. I was a fan of [the Once philosophy DHH was pushing a couple of years ago](https://world.hey.com/dhh/campfire-is-once-1-d2cebd12), and I don't see how a subscription is defensible in this space. The underlying models keep getting better and cheaper every quarter; charging a recurring fee for the interface on top, while taking away the user's choice of which models run, doesn't add up.

BYOK (bring your own key) may sound like extra cost and friction to many, but in practice it's neither. My Gemini bill comes out to under 10 cents a month, and ElevenLabs runs me roughly $3 a month at daily-use volume. The real payoff of holding the keys, though, is the freedom to swap. I've already moved VoiceInk's transcription provider three times this past year (local Whisper, then Groq, now ElevenLabs), without ever waiting on the app to catch up to whatever had shipped most recently.

The other thing you get with VoiceInk is a developer you can reach. When VoiceInk's "pause media while recording" feature (pauses Spotify and the like while you're speaking and resumes after) misbehaved on my setup, it was acknowledged and fixed quickly. When [OpenAI's realtime Whisper shipped last week](https://openai.com/index/advancing-voice-intelligence-with-new-models-in-the-api/) and VoiceInk didn't yet support it, I [opened an issue](https://github.com/Beingpax/VoiceInk/issues/694) on the repo and expect it to be resolved any day. That kind of turnaround is harder to get once a product serves millions and is owned by a large team with a multi-year roadmap and a board (though a one-person shop has bottlenecks of its own, granted).

[Wispr Flow has raised $81M](https://www.prnewswire.com/news-releases/wispr-raises-25m-to-build-its-voice-operating-system-302621858.html) to build a "voice operating system," which is a different bet on a different timeline. For what dictation actually is today, though, I honestly don't see what $10 to $15 a month for a subscription buys me over $25 once plus a few dollars a month in API fees.
