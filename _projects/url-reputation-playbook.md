---
layout: project
title: "SOAR Reputation Playbooks (URL & IP)"
date: 2026-01-07
categories: [projects]
tags: [SOAR, automation, threat-intel, splunk, python, incident-response]
description: >
  A unified set of Splunk SOAR playbooks that automate URL and IP reputation investigations using layered enrichment, internal telemetry, and analyst-readable case notes.
image:
  path: https://github.com/sparks-cam/sparks-cam.github.io/releases/download/assets-v1/url_playbook_overview_obfuscated.webp
links:
  - title: GitHub Repo (sanitized examples)
    url: https://github.com/yourusername/soar-hardening
---

## SOAR Reputation Playbooks (URL & IP)

## About This Project

Reputation-based alerts make up a large portion of day-to-day Incident Response work. Suspicious URLs from phishing emails. IPs from firewall logs, proxy traffic, IDS alerts, and cloud telemetry.

Individually, these investigations are simple. At scale, they become repetitive, inconsistent, and slow.

I built these Splunk SOAR playbooks to standardize how URL and IP investigations are performed, reduce analyst fatigue, and surface useful context quickly without forcing analysts to pivot across tools or interpret raw data.

This project consists of two playbooks built on the same core design:
- URL Reputation Playbook
- IP Reputation Playbook

## Shared Playbook Design

Both playbooks follow the same high-level workflow:

1. Validate indicator input and fail fast
2. Fan out enrichment across multiple services
3. Pull internal Splunk telemetry for real-world context
4. Normalize outputs into readable markdown tables
5. Attach structured notes directly to the case

Each enrichment path runs independently and writes results as they become available.

# URL Reputation Playbook

## Purpose

The URL Reputation playbook focuses on phishing and web-based threats and quickly answers whether a URL is malicious, suspicious, or safe.

## URL Enrichment Sources

- URL analysis and detonation
- Phishing reputation services
- Malware analytics and sandbox detonation
- WHOIS enrichment
- Website screenshots
- TIP sandbox detonation
- Internal Splunk web and proxy searches

## URL Playbook – Input Validation & Fan-Out

```python
@phantom.playbook_block()
def null_input_decision(...):
    found_match_1 = phantom.decision(
        container=container,
        conditions=[["playbook_input:domain", "not in", None]],
        delimiter=None
    )

    if found_match_1:
        url_analysis_detonate(...)
        domain_screenshot(...)
        whois_detonate(...)
        malware_analytics_url(...)
        url_protocol_format(...)
        web_splunk_query(...)
        tip_sandbox_detonate(...)
        return
```

## URL Playbook – URL Normalization

```python
@phantom.playbook_block()
def url_protocol_format(...):
    template = "http://{0}\n"
    parameters = ["playbook_input:domain"]
    phantom.format(
        container=container,
        template=template,
        parameters=parameters,
        name="url_protocol_format"
    )

    malware_sandbox_detonate(container=container)
    phish_reputation_check(container=container)
```

## URL Playbook – Analyst Note Formatting

```python
template = """### URL Analysis Results

Field | Value
--- | ---
Indicator | {0}
Verdict | {1}
Screenshot URL | {2}
ASN | {3}
Country | {4}
"""
```

# IP Reputation Playbook

## Purpose

The IP Reputation playbook focuses on network-based threats and infrastructure analysis.

## IP Enrichment Sources

- IP abuse and reputation services
- Multi-source reputation intelligence
- WHOIS and ASN lookups
- Secure Web Gateway lookups
- Internet exposure scanning
- URL/IP analysis and screenshots
- Internal Splunk firewall and traffic searches
- TIP enrichment via REST searches

## IP Playbook – Input Validation & Fan-Out

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

## IP Playbook – External Reputation Enrichment

```python
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

## IP Playbook – Internal Splunk Context

```python
@phantom.playbook_block()
def splunk_ip_search(...):
    phantom.act(
        "run query",
        parameters=parameters,
        assets=[ASSET_SPLUNK_SEARCH],
        callback=splunk_ip_format
    )
```

## IP Playbook – Analyst Note Formatting

```python
template = """### IP Reputation Results

Field | Value
--- | ---
Indicator | {0}
ISP | {1}
Country | {2}
Total Reports | {3}
Usage Type | {4}
"""
```

## What Analysts See

- External reputation verdicts
- Infrastructure and ASN context
- Internal traffic evidence
- Internet exposure indicators (IP)
- Sandbox and screenshot context (URL)
- Clean, readable case notes

## Key Features

- URL and IP reputation automation
- Multi-source enrichment
- Internal Splunk telemetry correlation
- TIP integration via REST searches
- Modular, reusable SOAR design
- Analyst-readable markdown notes
- Safe for enterprise and MSSP use

## Future Improvements

- Shared enrichment modules across indicators
- Cross-indicator correlation in a single case
- Risk scoring based on multiple services
- Optional automated blocking with analyst approval
- Expansion to domain and file hash reputation
