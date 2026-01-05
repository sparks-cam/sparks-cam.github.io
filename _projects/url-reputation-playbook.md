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

In my experience when it comes to IR, repetitive enrichment tasks especially around suspicious URLs consumed a surprising amount of time. I built this Splunk SOAR playbook to **reduce alert fatigue**, **speed up decision-making**, and **increase investigation consistency** across our SOC and our MSSP.

This playbook automatically detonates URLs against multiple internal and external reputation sources, normalizes the results, and **recommends next steps** aligned to our security policies.

> **Impact:** Reduced manual URL triage time while improving evidence quality in cases.

---

## Problem & Motivation

When analysts are processing dozens or hundreds of alerts per shift:

- Manual URL checks become slow and frustrating  
- Results may vary based on human interpretation  
- Analyst notes often lack standardized context  
- Adversaries move faster than humans can click

---

## How It Works

**Input methods:**

- Extracted automatically from SOAR artifacts (email, phishing alerts, web events)
- Analyst-prompted URL submission

**Automated workflow:**

1. Validate the indicator (fail fast on null/empty input)
2. Fan-out enrichment to multiple sources (URL analysis, phishing rep, malware analytics/sandbox, WHOIS, screenshots, TIP sandbox)
3. Run internal context searches in Splunk (Web data model + TIP REST enrichment panels)
4. Write **consistent markdown notes** per enrichment so the case tells the story without extra clicks
5. Join results back together and publish a final **one-glance verdict table**

The result is a **single actionable summary** inside the case.

---

## Architecture & Services

At a high level, the playbook fans out enrichment against multiple services, then joins results back together into a **single verdict table** and **human-readable notes**.

**Core actions/services (sanitized placeholders):**

- `ASSET_URLSCAN` — URL analysis service (detonate + report)
- `ASSET_WEB_SCREENSHOT` — Website screenshot capture (vault)
- `ASSET_WHOIS` — WHOIS enrichment
- `ASSET_PHISH_REP` — Phishing/URL reputation service
- `ASSET_MALWARE_ANALYTICS` — Malware analytics / URL detonation
- `ASSET_MALWARE_SANDBOX` — Malware sandbox (URL detonation + report)
- `ASSET_TIP_SANDBOX` — TIP sandbox (detonate + fetch report)
- `ASSET_SPLUNK_SEARCH` — Splunk search asset (Web data model + TIP REST enrichment)

This design keeps everything **modular** (each enrichment writes its own note), and the end result is **correlated** into one summary table for quick analyst decisions.

---

## Code Snippets

Below are trimmed excerpts from the sanitized playbook code. They show the *actual* SOAR patterns used: decision gating, fan-out enrichment, consistent markdown notes, and a final correlated verdict.

### 1) Sanitized assets/config (safe to share)

```python
# SANITIZED & NORMALIZED VERSION (public shareable)
# Map these placeholders to your local Splunk SOAR asset names/URLs.
ASSET_URLSCAN = "<url_analysis_asset>"
ASSET_WEB_SCREENSHOT = "<web_screenshot_asset>"
ASSET_WHOIS = "<whois_asset>"
ASSET_PHISH_REP = "<phishing_reputation_asset>"
ASSET_MALWARE_ANALYTICS = "<malware_analytics_asset>"
ASSET_MALWARE_SANDBOX = "<malware_sandbox_asset>"
ASSET_TIP_SANDBOX = "<tip_sandbox_asset>"
ASSET_SPLUNK_SEARCH = "<splunk_search_asset>"

SPLUNK_CLOUD_BASE = "https://<your_splunk_cloud_stack>"
TIP_APP_NAME = "<TIP_app_name>"
TIP_SANDBOX_BASE_URL = "https://<tip_sandbox_base_url>"
```

### 2) Input validation + fan-out enrichment

The playbook gates execution so we don’t run a bunch of actions on a null/empty indicator, then kicks off parallel enrichment blocks.

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
        splunk_rf_search(...)
        web_splunk_query(...)
        tip_sandbox_detonate(...)
        return

    null_fail_format(...)
```

### 3) URL normalization for downstream reputation/sandbox actions

Some actions expect a fully-qualified URL. This helper formats the input and immediately triggers the blocks that depend on it.

```python
@phantom.playbook_block()
def url_protocol_format(...):
    template = "http://{0}
"
    parameters = ["playbook_input:domain"]
    phantom.format(container=container, template=template, parameters=parameters, name="url_protocol_format")

    malware_sandbox_detonate(container=container)
    phish_reputation_check(container=container)
```

### 4) Consistent analyst notes (example: URL analysis report)

A pattern used throughout: *collect results → build a markdown table → add a case note.*

```python
@phantom.playbook_block()
def format_url_analysis_note(...):
    template = """### URL Analysis Results

Field | Value
--- | ---
Indicator | {0}
Action Status | {1}
Report Status | {2}
Verdict | {3}
Screenshot URL | {4}
Response IP | {5}
GeoIP Country Code | {6}
GeoIP Country | {7}
GeoIP Timezone | {8}
ASN Name | {9}
ASN Description | {10}
"""

    parameters = [
        "playbook_input:domain",
        "url_analysis_detonate:action_result.status",
        "url_analysis_report:action_result.status",
        "url_analysis_report:action_result.data.*.verdicts.urlscan.malicious",
        "url_analysis_report:action_result.data.*.task.screenshotURL",
        "url_analysis_report:action_result.data.*.data.requests.0.response.asn.ip",
        "url_analysis_report:action_result.data.*.data.requests.0.response.geoip.country",
        "url_analysis_report:action_result.data.*.data.requests.0.response.geoip.country_name",
        "url_analysis_report:action_result.data.*.data.requests.*.response.geoip.timezone",
        "url_analysis_report:action_result.data.*.data.requests.*.response.asn.name",
        "url_analysis_report:action_result.data.*.data.requests.*.response.asn.description",
    ]
    phantom.format(container=container, template=template, parameters=parameters, name="format_url_analysis_note")
```

### 5) Final correlated verdict table (the “one-glance” summary)

Once the key actions finish, the playbook joins results and writes a single table showing verdict/score per source.

```python
@phantom.playbook_block()
def join_filter_matching_domains(...):
    if phantom.completed(action_names=[
        "url_analysis_detonate",
        "malware_sandbox_detonate",
        "whois_detonate",
        "phish_reputation_check",
        "malware_analytics_url",
        "tip_sandbox_report"
    ]):
        filter_matching_domains(container=container, handle=handle)

@phantom.playbook_block()
def format_verdict_note(...):
    template = """Indicator | URL Analysis | Malware Sandbox | Phish Reputation | Malware Analytics | TIP Sandbox
---|---|---|---|---|---
%%
{0} | {1} | {2} | {3} | {4} | {5}
%%"""

    parameters = [
        "playbook_input:domain",
        "filtered-data:filter_matching_domains:condition_1:url_analysis_detonate:action_result.data.*.verdicts.urlscan.malicious",
        "filtered-data:filter_matching_domains:condition_1:malware_sandbox_detonate:action_result.data.*.result.report.verdict",
        "filtered-data:filter_matching_domains:condition_1:phish_reputation_check:action_result.message",
        "filtered-data:filter_matching_domains:condition_1:malware_analytics_url:action_result.data.*.threat.score",
        "filtered-data:filter_matching_domains:condition_1:tip_sandbox_report:action_result.data.*.report.summary.score",
    ]
    phantom.format(container=container, template=template, parameters=parameters, name="format_verdict_note")
```


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


