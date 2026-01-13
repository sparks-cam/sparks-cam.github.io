---
layout: project
title: "SOAR IP Reputation Playbook"
date: 2026-01-07
categories: [projects]
tags: [SOAR, automation, threat-intel, splunk, python, incident-response]
description: >
  Splunk SOAR playbook that automates IP reputation investigations using layered enrichment, internal telemetry, and analyst-readable case notes.
image:
  path: https://github.com/sparks-cam/sparks-cam.github.io/releases/download/assets-v1/url_playbook_overview_obfuscated.webp
links:
  - title: GitHub Repo (sanitized example)
    url: https://github.com/yourusername/soar-hardening
---

## Splunk SOAR IP Reputation Playbook

## About This Project

In Incident Response, IP addresses show up everywhere. Firewall alerts, proxy logs, IDS signatures, endpoint detections, and cloud telemetry all eventually boil down to “what is this IP and should I care?”

Manually investigating IPs across multiple tools quickly turns into context switching, copy-pasting, and inconsistent documentation. I built this Splunk SOAR playbook to **standardize IP investigations**, **reduce analyst workload**, and **surface useful context fast enough to support real decisions**.

This playbook automates enrichment across multiple reputation sources, internal Splunk telemetry, and threat intelligence platforms, then writes everything back into the case in a format analysts can actually scan and understand.

> **Impact:** Faster IP triage with more consistent investigations and significantly cleaner case notes.

---

## Problem & Motivation

A typical manual IP investigation usually looks like this:

- Check one or more external reputation services
- Look up WHOIS and ASN ownership
- Search firewall or proxy logs for activity
- Check threat intel platforms for sightings
- Manually summarize findings into notes

That process is slow, varies by analyst, and makes it easy to miss important context under pressure.

The goal of this playbook was to turn that workflow into **one repeatable, automated investigation path**.

---

## How It Works

**Input methods:**
- Playbook input IP
- Artifact-based IP extraction from alerts and notables

**Automated workflow:**
1. Validate the IP input (fail fast if missing)
2. Fan out enrichment across external reputation and intel services
3. Pull internal context using Splunk searches
4. Normalize outputs into consistent markdown tables
5. Attach analyst-readable notes directly to the case

All enrichment actions run in parallel where possible and write results as they complete.

---

## Architecture & Services

At a high level, this playbook performs **layered enrichment** across reputation, infrastructure, and observed activity.

**Core actions and services (sanitized placeholders):**
- `ASSET_IP_REP` — IP abuse / reputation service
- `ASSET_REP_SERVICE` — Multi-source reputation service
- `ASSET_WHOIS` — WHOIS lookup
- `ASSET_URL_ANALYSIS` — URL/IP analysis and screenshots
- `ASSET_SWG` — Secure Web Gateway lookups
- `ASSET_INTERNET_EXPOSURE` — Internet exposure scanning
- `ASSET_SPLUNK_SEARCH` — Internal Splunk searches (firewall, proxy, threat data)
- `TIP REST Search` — Threat Intelligence Platform enrichment via REST queries

Each service is modular and writes its own note. No giant JSON blobs dumped into cases.

---

## Code Highlights

Below are trimmed excerpts from the sanitized playbook showing real SOAR patterns used in production-style workflows.

### 1) Input validation and fan-out enrichment

```python
@phantom.playbook_block()
def decision_1(...):
    found_match_1 = phantom.decision(
        container=container,
        conditions=[["playbook_input:ip", "not in", None]],
        delimiter=None
    )

    if found_match_1:
        ip_abuse_reputation(...)
        whois_ip(...)
        swg_ip_lookup(...)
        reputation_service_ip(...)
        tip_ip_search(...)
        url_analysis_lookup(...)
        splunk_ip_search(...)
        internet_exposure_ip(...)
        return
```

This ensures enrichment only runs when a valid IP is present and allows all services to execute in parallel

### 2) External reputation enrichment (example)

```
@phantom.playbook_block()
def ip_abuse_reputation(...):
    parameters.append({
        "ip": playbook_input_ip_item[0],
        "days": 10,
    })
    phantom.act(
        "lookup ip",
        parameters=parameters,
        assets=[ASSET_IP_REP],
        callback=ip_abuse_reputation_format
    )
```

### 3) Internal Splunk telemetry search

```
@phantom.playbook_block()
def splunk_ip_search(...):
    phantom.act(
        "run query",
        parameters=parameters,
        assets=[ASSET_SPLUNK_SEARCH],
        callback=splunk_ip_format
    )
```

This pulls firewall and traffic context so analysts can see actual observed activity, not just reputation scores

###  4) Analyst-friendly markdown notes

Each enrichment path formats results into clean markdown tables before attaching them to the case.

```
template = """### IP Reputation Results

Field | Value
--- | ---
Indicator | {0}
ISP | {5}
Country | {10}
Total Reports | {12}
Usage Type | {13}
Latest Comment | {14}
"""
```

The focus is fast scanning and decision support, not raw data dumps.
