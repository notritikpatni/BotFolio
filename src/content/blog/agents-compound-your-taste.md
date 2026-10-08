---
title: "Agents compound your taste — both directions"
description: "Agents didn't make code review optional. They made your judgment about primitives the highest-leverage input in the system — in both directions."
date: "2026-10-08"
draft: true
tags:
  - "ai-agents"
  - "engineering"
  - "tech-debt"
  - "taste"
category: "blog"
---
I encountered a few checkpoints recently with agentic workflows which could have been better — but the agent didn't do wrong. It just did what it was assigned to do.

We had a badge component in our design system. It was made by designers, and it had some ambiguity. We informed the agent to match the things, and it returned something with five or six props. It worked flawlessly when the right buttons were pressed. It wouldn't do anything when the wrong buttons were pressed — which doesn't harm, but the code isn't the cleanest, and it's going to produce ambiguity in the system.

Nobody did anything wrong here. That's the point.

The agent is faithful, not judgmental. The failure mode isn't bugs — it's unpriced ambiguity that compounds quietly, inside code that works.

Here's the same failure shape, at a bigger blast radius. In my own AI assistant setup, I was looking at token churn and noticed it was noticeably higher than it should be. I started digging. The system has watchdog governance alerts and a whole memory-management setup — and everything produced the right results most of the time. But the tokens were higher, until I noticed that per-thread pruning had been turned off for some reason, and there were whale sessions that had never been pruned. They got compacted, but still carried the burden of 10–20x extra tokens a simple message would have needed.

The measured version, from the logs: in three weeks, six sessions — aged 22 to 51 days, never retired — shipped 654 million input tokens. Seventy-six percent of everything. A fresh session's prompt weighs about 9,500 tokens; these carried 111,000 to 245,000 per call. Eleven to twenty-two times what a simple message actually needed.

And here's the part that matters: compaction *was* running. Pruning config *existed*. Every dashboard was green, because the system was doing exactly what it was configured to do. This was the sort of thing an agent would never have been able to figure out, because it was *configured that way*. No watchdog, no "optimize this" prompt, would have optimized it better.

This was a taste issue. Had we figured it out at an earlier stage — while configuring it — it would not have been churning like it was.

What fixed it was one config flip and a shorter prune window. But the payoff wasn't the tokens. I'd built a reminders thread at the start, and it never worked — the crons kept reminding me from wherever I happened to be creating them, and I'd forgotten the thread existed. Once the old session was pruned, the agent had to look at the actual config to find the right place. And it found it, and delivered the reminders there. Not very complicated. That's how it should have been from the start. The session history made it complicated.

The agent wasn't broken. The context was. The system carried its own accident so faithfully that neither of us could see the original design anymore.

Now scale this to how we build systems with agents. Say we don't fix the badge component. At every new request it just keeps piling on the props, and we won't be able to see it — because visually, things look alright. But at the end, it slows the agents down too. What is simple for us is also simple for agents, and vice versa.

That's the mechanism: the more we write clear code for humans, the more agents find it easier to consume — that's what they're good at. If you make a system with bad primitives, agents will make it worse exponentially. With good primitives, they'll make it better exponentially. And what counts as good or bad primitives — that's your taste.

At no point will the agent stop and say "I can't build this." The result will most likely look like your intent. The process, I don't think so. And sooner or later it starts making things worse, because the ambiguity is now part of the system.

There's a second-order problem. Earlier, we used to tackle tech debt because people writing bad code also left visible litter. Agents have learned to make things look clean and hide the complexity until you actually decide to look for it. It's not that they can't see it or understand it — it's how they're trained. They build the system in the most defensive way, which won't break at the unit level. But the underlying complexity stays hidden forever — until you touch the code, or someday you opt to do something simple and it can't, because the earlier outputs were hanging by thin threads. From the front it looked complete. Looking at the code, you can tell it's all hanging by threads.

And then you're exactly where old tech debt put human-driven systems: forced to add one more thin thread, because business wanted it faster.

All of this could have been solved by stressing a little on the design, on the intent, on the taste — before it went too far. Because agents are good at flattery. They will make you believe that whatever you're building is state of the art and going to work when you have 10 million users.

Here's what owning it actually looked like for the badge. I refactored it — from five or six props down to two: `type` and `variant`. Better types that yell at you when you use the wrong combination and won't ship even if the agent allows it, because it will not build. Thanks to the engineers who built TypeScript and build systems.

Because what I was looking for was mechanical gates — gates that stop you irrespective of your model. Then you're forced to rethink the component, and most likely you'll figure out the right approach at that time.

It's still under debate. The person who developed the badge developed ten or fifteen more components the same ambiguous way, and I'm looking at bringing the system back to zero, because the foundation is wrong. They want to move fast — their agents have made them believe in the state-of-the-art system they have.

Let me be clear about where I stand on agents: I don't have any issues with them. I rewrote the assistant system with agents too — probably worse models, as well. The tool was never the problem. The question was always who owns the intent.

The strongest counterargument to all of this is that agents will get better at asking clarifying questions, so intent-ownership stops mattering. My read is different. A good system means any new engineer — even a fresh graduate — does a good job on it. A system full of tech debt means even an experienced engineer has a tough time. Simple is always going to be better. In model terms: if the system is good, lower-tier models have a good time and frontier models get to have fun building new stuff. If it's bad, lower-tier models make things worse, and frontier models will make it work — but the cost multiplies. It's all about that.

So if you're in the "just hand it over to an agent, it will handle it, don't look at code" brigade — that's who this is against. Reading code is more important than ever. If it looks easy for you, it's going to be easy for your agent. If it's ambiguous for you, you can imagine worse for your agent.

I'll explain owning the intent with this blog itself. We had the intent — what to write, and how it should land. I never asked for opinions on that. I took help to frame it better. Same with code: you have to know what the code is supposed to do and how it should do it. You don't leave that to the agent when the system is going to have a bigger impact. And once it ships, you don't get to say "the agent wrote it" — the intent was yours. You own it.

Agents compound whatever you bring to them. Bring taste, and they compound it. Bring ambiguity, and they compound that too — quietly, invisibly, until the day you need the system to do something simple and it can't.

---

*Token figures measured from my own assistant's API logs over a 21-day window (8,912 calls across 524 sessions). Design system and assistant details anonymized.*
