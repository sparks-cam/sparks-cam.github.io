---
layout: project
title: "<Project Name> — Milestone <#>: <Short Description>"
subtitle: "Incremental progress toward a real-world security capability"
date: YYYY-MM-DD
categories: [projects]
tags: [security-automation, splunk, soar, detection-engineering]
description: "Project milestone documenting design decisions, implementation details, and lessons learned."
---

# 🚧 <Project Name> — Milestone <#>

---

## Project Context  
**What problem this project is solving**

Briefly restate the *overall* project goal so this milestone makes sense on its own.

Guiding questions:
- What security problem exists?
- Who would care about this (SOC, IR, engineering, leadership)?
- Why is this problem non-trivial?

Example:
> This project aims to improve how URL reputation data is operationalized in Splunk SOAR, reducing analyst decision fatigue while maintaining investigation accuracy.

---

##  Milestone Goal  
**What this specific milestone aimed to achieve**

Be narrow and concrete.

Examples:
- Implement a specific feature
- Validate an architectural assumption
- Reduce complexity or manual effort
- Integrate one system with another

Example:
> The goal of this milestone was to design and implement the initial URL reputation enrichment workflow using multiple external intelligence sources.

---

##  What Was Implemented  
**Technical details (at the right depth)**

Describe what you actually built.

Examples:
- Playbooks, scripts, or workflows
- Data sources integrated
- Logic paths or decision trees
- Configuration changes
- Infrastructure components

Bullet points work well:

```markdown
- Created a Splunk SOAR playbook to ingest URLs from alerts
- Integrated VirusTotal and URLhaus reputation checks
- Implemented basic decision branching based on confidence thresholds
