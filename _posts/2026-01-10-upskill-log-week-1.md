---
title: "Upskill Journey – Week 1: Admitting I’m Lost (and Why That’s Probably Fine)"
date: 2026-01-10
tags: [cybersecurity, upskill, incident-response, ai, career]
---

I almost didn’t write this post.

Not because I didn’t do anything this week — but because most of what I did didn’t *feel* impressive. No shiny new tool. No model training. No “look at this cool AI thing I built.” Just a lot of thinking, sketching, and honestly… realizing how overwhelmed I’ve been trying to figure out what the *right* direction even is anymore.

So this post is Week 1. And Week 1 is mostly about admitting I’m lost.

## The problem I’m actually dealing with

I work in Incident Response. I write detections. I design SOAR playbooks. I triage alerts. On paper, that should feel stable. But the industry noise right now is wild.

Every other post is:
- “AI will replace SOC analysts”
- “Everything will be automated”
- “Learn ML or you’re doomed”

And I think the most dangerous thing about that noise isn’t that it’s wrong — it’s that it pushes you toward **learning random things with no proof of value**.

This week I stopped asking:
> “What should I learn next?”

And started asking:
> “What do I do today that AI *cannot safely own alone*?”

That question changed everything.

## What I did this week (actually)

No heroics. No late nights. Just consistent, boring work.

### 1. I looked at alerts differently
Instead of asking “is this a true positive?”, I started asking:
- What context is *missing* here?
- What would an LLM confidently guess wrong?
- Where does automation need to stop and escalate?

That alone surfaced more useful insights than any AI course could have.

### 2. I sketched decision points, not tools
I didn’t write code. I wrote logic:
- If X + Y + Z → auto close
- If privilege escalation + uncertainty → human review
- If blast radius unknown → stop automation

This felt obvious… which probably means it’s important.

### 3. I started documenting *judgment*
Not detections.
Not alerts.
Judgment.

Why something *shouldn’t* be automated.
Why confidence matters more than speed.
Why false certainty is worse than delay.

That’s not something I’ve ever seen clearly articulated in most SOC tooling.

## A realization I didn’t expect

I kept thinking I needed to “learn AI.”

What I actually need is to **design systems where AI is constrained**.

AI is good at summarizing.
AI is bad at understanding consequences.
AI is dangerous when it *sounds* confident.

That means the value isn’t in prompting better — it’s in deciding **where prompts are even allowed to exist**.

That’s an Incident Response problem. Not an ML one.

## What this week changed for me

I’m no longer chasing:
- generic ML learning paths
- random AI certifications
- “SOC chatbot” projects

Instead, I’m focusing on:
- decision frameworks
- escalation logic
- accountability boundaries
- automation with brakes

That feels a lot more grounded — and honestly, more aligned with how real incidents actually work.

## What’s coming next (Week 2)

Next week I’m going to:
- Start formalizing an **AI-aware incident triage framework**
- Map common alerts to “AI allowed / AI forbidden”
- Document failure cases where automation would cause real damage

No promises it’ll be pretty yet. But it’ll be real.

If nothing else, Week 1 proved something important:

I don’t need to become someone else to stay employable.  
I need to make what I already do **explicit, defensible, and harder to automate incorrectly**.

That feels like a direction I can actually walk.
