---
layout: project
title: "Defining Secure Automation: A Detection - SOAR Contract"
date: 2026-02-01
categories: [projects]
tags: [soar,security-automation,detection-engineering,governance, ai,incident-response]
description: >
  A vendor-neutral contract that governs how security detections are allowed to trigger automated response, with clear safety boundaries, human approval points, and auditability.
image:
  path: https://github.com/sparks-cam/sparks-cam.github.io/releases/download/assets-v1/detection_soar_contract.webp
---


# Defining Secure Automation: A Detection - SOAR Contract

## Purpose 
This document defines **what automated actions are allowed when a security detection fires**, under what conditions, and when a human must be involved.

Its purpose is to:
- Prevent unsafe automation
- Reduce response time responsibly
- Ensure accountability during incidents
- Preserve evidence before any destructive action

This contract acts as a **safety boundary** between detection logic and automated response. This contract acts as a vendor-neutral, standards-aligned reference implementation for governing automated security response.

---
## Scope
This contract applies to **any detection that triggers automated or semi-automated actions** via SOAR.

If a detection does **not** meet this contract, it **must not** trigger automation.

---
## Core Principle
> Automation may act **only within explicitly approved boundaries**.  
> Anything ambiguous, irreversible, or high-impact requires a human decision.

---
## Reference Framework Alignment

This Detection → SOAR Contract is informed by established cybersecurity and AI risk management frameworks.  
It operationalizes high-level guidance into **concrete, enforceable automation controls**.

The intent is not compliance for its own sake, but **risk-aware, auditable automation** that aligns with widely accepted standards.

---
### NIST Cybersecurity Framework (CSF 2.0)

[NIST.CSWP.29.pdf](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf "https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf")

This contract directly supports the following NIST CSF 2.0 functions:
#### GOVERN
- Defines **decision authority** for automated actions
- Establishes **accountability** for automation outcomes
- Sets **risk-based boundaries** on autonomous response
- Requires documentation and review of automation behavior
#### DETECT
- Requires detections to declare **confidence, scope, and intent**
- Prevents ambiguous signals from triggering unsafe actions

#### RESPOND
- Enforces **controlled response actions**
- Requires escalation thresholds and human approval when risk is unclear
- Mandates evidence preservation prior to containment

#### RECOVER
- Requires **rollback capability** for automated actions
- Supports rapid recovery when automation misfires

**Summary:**  
This contract translates CSF principles into enforceable response controls, ensuring automation operates within defined governance boundaries.

---
### NIST SP 800-61 (Computer Security Incident Handling Guide)

This contract operationalizes key guidance from NIST SP 800-61 by ensuring:

- Incident response actions are **deliberate and documented**
- Escalation decisions are **risk-based**, not automatic
- **Evidence integrity** is preserved prior to containment
- Human judgment is retained for high-impact or irreversible actions

Key alignments include:
- Detection confidence tiers → Incident categorization
- Human-in-the-loop requirements → Escalation control
- Evidence preservation → Forensic readiness
- Rollback logic → Containment safety

**Summary:**  
The contract ensures automated response does not bypass core incident-handling principles required for reliable investigation and recovery.

---
### MITRE ATT&CK (Supporting, Non-Authoritative)

MITRE ATT&CK is referenced **only** to:
- Justify detection intent
- Explain behavior-based confidence scoring

MITRE ATT&CK **does not authorize response actions**.

Response authority is governed exclusively by:
- Confidence thresholds
- Risk tier classification
- Human approval requirements defined in this contract

**Summary:**  
ATT&CK informs *what is being detected*, not *what actions are taken*.

---
### NIST AI Risk Management Framework (AI RMF 1.0)

This contract is designed to be directly applicable to **AI-assisted or AI-driven response mechanisms**. 

Reference Link:
[NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework)

It aligns with the NIST AI RMF functions as follows:
#### GOVERN
- Establishes **human accountability** for automated decisions
- Defines **autonomy boundaries** for AI-enabled response
- Requires periodic review of automated decision systems
#### MAP
- Identifies where automation or AI is used in the response lifecycle
- Classifies risk based on system impact and uncertainty
#### MEASURE
- Requires logging of:
  - Decision inputs
  - Confidence levels
  - Actions taken
  - Outcomes and rollbacks
#### MANAGE
- Enforces limits on automated actions
- Requires human approval for high-risk decisions
- Provides rollback and recovery mechanisms

**Summary:**  
This contract functions as a **governance layer** that constrains AI autonomy, ensures auditability, and preserves human control over security decisions.

---
### Applicability to AI Agents and Autonomous Response

Any AI-driven or agentic response system triggering security actions must adhere to this contract.

This includes requirements to:
- Declare confidence and uncertainty
- Operate within approved autonomy levels
- Preserve evidence prior to action
- Support human override
- Produce auditable decision logs

Automation without governance creates risk.  
Automation with governance creates resilience.

---
## Required Detection Fields
Every automation-eligible detection **must supply** the following fields:

| Field | Description |
|-----|------------|
| `detection_id` | Unique identifier for the detection |
| `confidence_score` | Numeric confidence (0.0 – 1.0) |
| `severity` | Informational / Low / Medium / High / Critical |
| `entity_type` | User / Host / Application / Cloud Resource |
| `entity_id` | Identifier of the affected entity |
| `environment` | Prod / Non-prod / Clinical / Corporate |
| `evidence_links` | Pointers to logs/artifacts |
| `timestamp` | Detection time |

❌ Missing fields = **no automation allowed**

---
## Confidence Scoring Contract
Detections must assign a **confidence score**, not just severity.

### Confidence Scale
| Score Range | Meaning |
|-----------|--------|
| 0.00 – 0.39 | Weak signal / contextual |
| 0.40 – 0.69 | Suspicious |
| 0.70 – 0.84 | Likely malicious |
| 0.85 – 1.00 | High confidence malicious |

Confidence scoring logic **must be documented** in the detection.

---
## Allowed Automation Actions by Confidence Tier

### Confidence < 0.40
**Automation Allowed**
- Enrichment only
- Tagging
- Case creation

**Automation Forbidden**
- Containment
- Blocking
- Account modification

---
### Confidence 0.40 – 0.69
**Automation Allowed**
- Enrichment
- Correlation
- Notification
- Case creation
- Human approval request

**Automation Forbidden**
- Automatic containment
- Credential revocation
- Resource isolation

---
### Confidence 0.70 – 0.84
**Automation Allowed**
- Enrichment
- Evidence collection
- Recommended actions (human-in-the-loop)
- Temporary containment *with approval*

**Automation Requires**
- Explicit human approval
- Logged decision

---

### Confidence ≥ 0.85
**Automation Allowed**
- Pre-approved containment actions
- Evidence preservation
- Emergency notifications

**Automation Requires**
- Post-action review
- Automatic rollback readiness

---
## Human-in-the-Loop Requirements
A human **must approve** automation if **any** of the following are true:

- Confidence < 0.85
- Entity is marked as:
  - Executive
  - Clinical system
  - Production-critical service
- Action is irreversible
- Detection is first-seen
- Conflicting signals exist

---
## Evidence Preservation Requirements
Before **any destructive or blocking action**, automation must:

- Capture relevant logs
- Preserve timestamps
- Record entity state
- Store evidence links in the case

If evidence capture fails → **automation stops**

---
## Automation Risk Classification
Each detection must be assigned a **risk tier**:

| Tier | Description |
|----|------------|
| Tier 0 | Enrichment-only |
| Tier 1 | Human-approved actions |
| Tier 2 | Limited autonomous actions |

Tier assignment must be documented.

---
## Rollback & Safety Controls
For any containment action, the playbook must define:

- Rollback method
- Rollback authority
- Rollback timeout
- Rollback logging

No rollback path = **no automation**

---
## Failure Conditions (Automation Must Abort)
Automation must stop immediately if:

- Required fields are missing
- Confidence score is undefined
- Evidence collection fails
- API calls return inconsistent results
- Entity identity cannot be verified

Automation failures must be logged.

---
## Audit & Logging Requirements
All automated decisions must log:

- Detection ID
- Confidence score
- Action taken
- Approval source (human/system)
- Timestamp
- Outcome
- Rollback status (if applicable)

Logs must be retained per security policy.

---
## Review & Change Management
- Contract reviewed quarterly
- Changes require security leadership approval
- All dependent playbooks must be reviewed after updates

---
## Contract Acceptance
By deploying automation tied to this detection, the owning team accepts responsibility for:

- Defining safe action boundaries
- Maintaining accuracy of confidence scoring
- Reviewing automation outcomes
- Responding to failures

---
## Summary
This contract ensures automation:
- Acts quickly when confidence is high
- Defers to humans when risk is ambiguous
- Preserves evidence
- Is auditable and reversible

Automation without contracts creates risk.  
Automation with contracts creates resilience.
