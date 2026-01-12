---
title: "Week 1 – Redefining My Role: From Writing Detections to Owning Automation"
date: 2026-01-10
categories: [cybersecurity, soar, detection-engineering, career]
tags: [cybersecurity, soar, detection-engineering, career, automation, ai-resilience]
---

This week I officially started a deliberate shift in how I think about my role in cybersecurity.

Not a new job.  
Not a new title.  
Not even a new tool.

A **reframing**.

I already spend most of my day writing detections (correlation searches in Splunk) and designing SOAR playbooks. On paper, that sounds pretty future-proof. In reality, I’ve been feeling an uncomfortable truth creep in over the last year:

> Writing detections and building playbooks is no longer the hard part.

The hard part is deciding **how far automation should be allowed to go** — and being accountable when it does.

---

## The problem I’m trying to solve

Most detections today are written like this:
- Fire an alert
- Let the SOC figure it out
- Maybe automate a response later

That model doesn’t scale.
And worse — it hides risk.

Because the moment you automate *anything*, you’re no longer just detecting threats.  
You’re making **decisions on behalf of the organization**.

And those decisions carry blast radius:
- Locking accounts
- Blocking access
- Preserving (or losing) evidence
- Triggering HR, legal, or patient-safety consequences

AI can help write detections.
AI can even suggest playbooks.

AI **cannot** own those outcomes.

---

## What I worked on this week

Instead of writing “another detection,” I focused on something I honestly hadn’t formalized before:

### A **Detection → SOAR Contract**

The idea is simple:
A detection shouldn’t just say *“something looks bad.”*
It should explicitly define:
- What confidence it has
- What actions are allowed
- What actions are forbidden
- When a human *must* be involved

This week I started drafting a contract that every automation-ready detection should meet.

So far, the contract defines:
- **Required fields** (confidence score, severity, asset criticality)
- **Allowed automation actions** by confidence tier
- **Escalation rules**
- **Evidence requirements** (what must be preserved before any action)

This immediately changed how I looked at my existing detections.

Some of them were solid.
Some of them were… automation landmines.

---

## The uncomfortable realization

I realized that a lot of detection engineering implicitly relies on *tribal knowledge*:

> “The SOC knows what to do with this alert.”

That’s fine when humans are always in the loop.
It’s dangerous when automation starts making decisions faster than people can think.

If I want to be valuable in an AI-heavy future, my value isn’t:
- Writing clever SPL
- Building flashy dashboards
- Chasing the next detection idea

My value is:
> Designing systems that **know when to stop**.

---

## What I produced (artifacts)

This week’s tangible outputs:
- First draft of a **Detection → SOAR Contract**
- One existing detection rewritten to comply with it
- Notes on where automation *must not* be allowed

Nothing flashy.
But foundational.

This is the kind of work that doesn’t show up in tool demos — and absolutely shows up during incidents.

---

## Why this matters for AI (and my career)

AI will keep getting better at:
- Suggesting detections
- Optimizing thresholds
- Generating playbooks

It will not:
- Understand organizational risk tolerance
- Be accountable when automation causes harm
- Decide when uncertainty is too high to act

That gap — the space between *capability* and *responsibility* — is where I’m deliberately positioning myself.

---

## Next week

Next up:
- Formalizing **SOAR engineering standards**
- Defining failure modes
- Making automation observable instead of magical

I want my automation to be boring, predictable, and defensible.
If it ever scares me, it’s wrong.

More to come.
