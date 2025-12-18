---
layout: project
title: "SOAR URL Reputation Playbook"
date: 2025-01-08
categories: [projects]
tags: [SOAR, automation, threat-intel, splunk, python, incident-response]
description: >
  Splunk SOAR playbook that automates URL reputation investigations with modular enrichment, analyst guidance, and case updates.
image:
  path: https://github.com/sparks-cam/sparks-cam.github.io/releases/download/assets-v1/url_playbook_overview_obfuscated.webp
links:
  - title: GitHub Repo (coming soon)
    url: https://github.com/yourusername/soar-hardening
---

## Splunk SOAR URL Reputation Playbook


## About This Project

As a cybersecurity engineer working in Incident Response at a children’s hospital, repetitive enrichment tasks — especially around suspicious URLs — consumed a surprising amount of analyst time. I built this Splunk SOAR playbook to **reduce alert fatigue**, **speed up decision-making**, and **increase investigation consistency** across our SOC.

This playbook automatically detonates URLs against multiple internal and external reputation sources, normalizes the results, and **recommends next steps** aligned to our security policies.

> **Impact:** Reduced manual URL triage time from ~6 minutes per IOC to < 45 seconds, while improving evidence quality in cases.

---

## Problem & Motivation

When analysts are processing dozens or hundreds of alerts per shift:

- Manual URL checks become slow and frustrating  
- Results may vary based on human interpretation  
- Analyst notes often lack standardized context  
- Adversaries move faster than humans can click

I wanted to build something that felt like:

> “Sysmon logic for URL triage — secure defaults, fast decisions, consistent outcomes.”

---

## How It Works

**Input methods:**

- Extracted automatically from SOAR artifacts (email, phishing alerts, web events)
- Analyst-prompted URL submission

**Automated workflow:**

1. Normalize URL (strip tracking parameters, canonicalize domain)
2. Query internal allow/block lists
3. Run threat intel reputation lookups
4. Enrich context (domain age, hosting provider, WHOIS API work in progress)
5. Score correlation + verdict
6. Update case artifacts, severity, and analyst notes

The result is a **single actionable summary** inside the case.

---

## Architecture & Services

- **Splunk SOAR** playbook built with:
  - Modular task design for reusability
  - Custom decision-block logic for malicious verdict correlation
- **Threat Intel APIs** (extensible):
  - Vendor reputation feeds
  - Internal reputation & policy engines
- **User-driven override logic**
  - Analysts can escalate false positives or suppress noisy URLs

---

## Screenshots

**Initial Input & Decision Path**

![Playbook Overview](https://github.com/sparks-cam/sparks-cam.github.io/releases/download/assets-v1/url_playbook_overview_obfuscated.webp)

---

## Key Features at a Glance

- Multi-source reputation scoring
- Automatically updates notable + artifacts
- Human-readable analyst note summary
- Designed as a **core module** for future playbooks
- Analyst override for case-by-case nuance
- Standardized enrichment → better case investigations

---

## Results & Lessons Learned

- Alert fatigue decreased on phishing URL queues  
- Junior analysts became more efficient with improved guidance  
- Case documentation quality improved significantly  
- Good foundation for AI-assisted triage and auto-blocking logic  

Building this taught me a ton about:

- Balancing **automation** with **analyst flexibility**
- Making security tooling feel like a **teammate**, not an obstacle
- The importance of **explainability** in automated decisions

---

## Roadmap & Future Capabilities

- Add **OpenAI Threat Analysis** summaries for explainability
- Add **GreyNoise** classification for scanning vs active threats
- Implement **kill chain scoring** to automatically escalate risky events
- Build correlation with **domain metadata APIs**
- Convert into a **shared library** for broader playbook use

---


