+++
title = "Waza: An AI Coach for Your Hands"
date = 2026-08-03
description = "Building an AI coach that sees what you see and talks you through real-world skills"

[taxonomies]
tags = ["waza"]

[extra]
stylesheets = ["css/video.css"]
social_media_card = "img/waza-ai-coach-for-your-hands-banner.webp"
+++

![A bright workshop with a white workbench holding a red toolbox, cordless drill, paint can, red furoshiki bundle, yarn, and chopsticks. On the wall, a hand-painted black patch carries the Waza logo as a neon sign: a white zigzag line from a camera lens to glowing red signal arcs.](/img/waza-ai-coach-for-your-hands-banner.webp)

Some of the loudest discussion today centers on if, and how fast, AI will automate human work away, and on how society should cope with that transition. Earlier this month, a bunch of economists, Nobel Prize winners, and tech execs signed a letter titled ["We Must Act Now"](https://www.businessinsider.com/ai-job-impact-read-letter-economists-executives-openai-anthropic-google-2026-7), warning of displacement at scale; last week, Jensen Huang fired back that AI ["kills tasks, not jobs"](https://fortune.com/2026/07/28/nvidia-jensen-huang-ai-killing-tasks-not-jobs/) and called the fear exactly backwards. Where the chips fall matters, and I don't claim to know who's right. What I do know is the past two months brought me to a corner of AI where there's much less to fear and a whole lot to gain: expanding what a person can confidently do with their own hands.

That confidence has been shrinking for a while. Knowledge work keeps us in front of screens for most of our waking hours, and we've responded by handing the physical side of life to specialists, one task at a time. Someone else cooks (a restaurant, a meal-prep service); a plumber comes over for an eight-minute job; a handyman hangs the shelf. Each handoff is reasonable on its own. Add them up, and our ability to deal with the physical world quietly atrophies, to the point where a dripping faucet looks like a phone call rather than the twenty-minute fix it often is.

It doesn't have to keep going this way. **Waza** (技, Japanese for technique) is my attempt at bending that curve back: an AI that sees what you see and coaches you through the task in front of you. This post walks through what it does today (demo videos included) and where I think it leads. If you use an iPhone and own a pair of Meta glasses, there's an invitation for you at the end.

## The Current Ceiling

Say you take on the faucet. Today's tech gives you two places to turn to, and each falls short in its own way. A chat assistant can talk you through the repair, but it's working blind: showing it your faucet means stopping to dry your hands and snap a photo, and the guidance comes back as words when what you really want is to be shown. YouTube has the exact opposite problem: it's all showing, by someone who knows what they're doing, but the video can't see *you*, so it can't tell you what you're doing wrong. Anyone who's rewatched the same fifteen seconds of a tutorial while their project sits in pieces knows that ceiling.

What I wish I had instead looks like the scene in *The Matrix* where Neo loads kung fu into his head (maybe minus the instant download into my motor cortex). You load a skill, and a coach watches you work, adapts to what you already know, and corrects you in pictures, not paragraphs, because humans absorb video far faster than text. That's the difference between watching someone change a tire and reading the glovebox manual.

A coach like that surely doesn't live in a chat window. It has to ride along, keeping your hands free and putting those pictures where your glance can catch them. The tech for that experience will likely arrive in stages: first, a phone propped next to the work, playing the clips; next, the same clips picture-in-picture on the lens of display glasses (the first generation is [already on shelves](https://www.meta.com/ai-glasses/meta-ray-ban-display-glasses-and-neural-band/)); and eventually, once full AR glasses reach the mass market, cues drawn right on top of whatever you're working on. Each stage moves the feedback closer to your eyes.

## What Waza Does Today

So, the question I set out to answer in June was: how much of this can be built right now? Quite a lot, it turns out. Waza is an iOS app that pairs with Meta glasses (or a phone in a pinch) and streams your first-person view to the AI coach, which talks you through each step of a skill and answers questions in about a second.

Every step in a skill carries a short point-of-view clip demoing the action to take, and the coach plays the right one at the right moment. Ask to see the step done, or fumble it, and the clip appears on your phone. The phone sits propped nearby; you glance over and keep both hands on the work. That's what separates a coach from a talking tutorial. The catalog holds a dozen skills to start, from tying a necktie to soldering wires to pan-searing a steak.

<div class="video-row">
  <video controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/chopsticks-hero-poster.jpg">
    <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/chopsticks-hero.mp4" type="video/mp4">
  </video>
  <video controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/necktie-hero-poster.jpg">
    <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/necktie-hero.mp4" type="video/mp4">
  </video>
  <video controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/furoshiki-hero-poster.jpg">
    <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/furoshiki-hero.mp4" type="video/mp4">
  </video>
</div>

(If you already own Meta glasses and want to try this yourself, feel free to skip ahead to the [invitation](#invitation).)

## Record Once, Coach Forever

The coach is only as useful as its skill catalog, so authoring needs to be easy. It turns out that modern video-understanding models can make it nearly automatic. To create a skill, you record yourself doing the task once, narrating as you go, and upload the footage. The pipeline breaks it into steps, trims each clip, zooms in on the key action, and writes the step-by-step coaching prompt. A ten-minute demo becomes a complete coached skill in about two and a half minutes, with nobody in the loop.

The friction this removes is key. A watchable YouTube tutorial has become a production: scripting, multiple takes, lighting, then hours in an editor like Final Cut Pro. That bar filters out most of the people who actually hold the skills. Waza asks for a single take and handles the edit itself. Your raw voice never ships (only your words guide the coach), so there's no performance to deliver either. Anyone who knows how to do something (cooking, DIY, handicrafts, a professional technique) can teach it by doing it once on camera.

<video class="video-solo" controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/record-skill-demo-v2-poster.jpg">
  <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/record-skill-demo-v2.mp4" type="video/mp4">
</video>

## Nothing to Tap

Apps still assume thumbs. Menus, search bars, detail pages: all built for tapping, and tapping is exactly what you can't do with wet, greasy, or occupied hands. In Waza, the coach drives the app too. Ask it to search the catalog, browse a category, or pull up a skill's page, and it happens on screen while your hands stay on the task. Load the skill, run the session, pause it, all by talking.

None of this was practical until recently. Voice models like OpenAI's GPT-Realtime-2.1 can now call tools mid-conversation, so the same model that's coaching you through a step can also press the app's buttons. Voice control stopped being the brittle "say 'menu' for options" experience long ago, becoming something closer to talking to a person who's holding your phone. I think it's one of the most slept-on changes in tech this year, and nowhere does it matter more than when both your hands are busy.

<video class="video-solo" controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/voice-navigation-hot-tub-poster.jpg">
  <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/voice-navigation-hot-tub.mp4" type="video/mp4">
</video>

## The Right Job for Camera Glasses

Camera glasses today don't have the best rep, and not without reason. The stories that travel are about wearers filming strangers who never agreed to be on camera, and that unease follows the category everywhere it goes, ever since Google Glass in 2013.

Coaching is the opposite kind of use. A session points the camera at the objects in your hands, on your own workbench or stove, and essentially never at another person. And the form factor fits the task like it was designed for it: your hands stay free, and the camera looks at the work from the same angle you do, following your attention as you move around it. One app won't repair the reputation of camera glasses, but uses like this one, over time, might.

## Where This Goes

The next step is finding out whether Waza is actually useful in the field, and where it falls short. Near term, that means: shipping features that smooth the experience (think asking the coach to add or fix a skill, or having it walk you through recording your own demo), and growing the catalog as people try skills, author their own, and pass them along. Every turn of that loop makes the coach more useful.

What I keep coming back to is what happens when users and skills reach critical mass. Skills are made of steps, and steps recombine. Picture an e-bike with a flat rear tire: getting to the tube means dealing with the motor cable first, and odds are no single skill covers that exact job yet. With a bike mechanic's flat-fix skill and an electronics hobbyist's cable-connector skill both in the catalog, though, the AI could splice steps from each into a repair guide nobody ever authored. That's **composability**, and it makes the catalog worth more than the sum of its skills: put the glasses on, say "I'd like to fix this", and a personalized skill assembles itself on the spot, with visual cues to match.

Scale changes the human side too. One setting I'd like to test is organized training: places that already teach hands-on skills, running sessions remotely, a human guide ready to step in whenever the AI hits a wall, and hundreds of people working through the same skill at once. The takeover part already works in Waza today (and is probably a post of its own).

That's a future I'd be excited to spend time building: more people (kids included) making and fixing real things in the real world, guided enough to succeed, instead of watching a screen while software acts on their behalf. Whether it's worth building, only users can tell me, and that's what the invitation below is for. We can't load kung fu yet, but coaching for everyone may already be within reach.

<a id="invitation"></a>

## Invitation

I'm putting together a small group of early testers: people excited by this kind of tech and comfortable with rough edges. It has to start small anyway, as Meta caps glasses testers at about a hundred per app until [publishing opens up in 2026](https://developers.meta.com/blog/introducing-meta-wearables-device-access-toolkit/). If you run an iPhone as your daily driver, own a pair of Meta glasses (Ray-Ban or Oakley), and have been wondering what to do with them beyond photos and music, [get in touch](https://vinidlidoo.github.io/contact/) and I'll set you up with a TestFlight build and a glasses invite. No glasses but curious? Here's the [store link](https://www.meta.com/ai-glasses/) (no commission or affiliation).

And if you're building or teaching anything adjacent (live guidance, egocentric video, skill capture, hands-on instruction of any kind), I'd love to compare notes. Same [contact page](https://vinidlidoo.github.io/contact/); tell me what you're working on.
