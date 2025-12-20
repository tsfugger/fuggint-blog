+++
author = "Tommy Fugger"
title = "Threat Intelligence 101"
date = "2025-12-17"
draft = true
description = "An Overview of Threat Intelligence"
categories = ["Threat Intelligence"]
tags = ["threat intelligence"]
image = "ThreatIntelligence101.png"
+++

## Overview

Threat Intelligence (TI) is the practice of collecting, analyzing, and operationalizing information about adversaries, their capabilities, infrastructure, and intent to inform security decision‑making. Where threat hunting is proactive detection inside your environment, threat intelligence provides the external context that helps you decide *what* to hunt for, *why* it matters, and *how* attackers are likely to operate.

A mature threat intelligence program does not exist to produce reports for their own sake — it exists to reduce risk, improve detection, and enable faster, more confident response.

## What is Threat Intelligence

### What It Is

- Evidence‑based insight about adversaries, campaigns, tools, techniques, and infrastructure
- Contextualized analysis that supports prevention, detection, response, and strategic planning
- Intelligence that is actionable, timely, and relevant to your organization

### What It Is Not

- A raw feed of IPs, hashes, or domains with no context
- A replacement for detection engineering, threat hunting, or incident response
- A one‑time project or a purely vendor‑driven capability

## Threat Intelligence Lifecycle

Like threat hunting, threat intelligence follows a repeatable, structured lifecycle to ensure consistency and impact.

**1. Planning and Direction**  
Intelligence analysts must first define *what* intelligence questions need answered. For example:  
- Which ransomware groups target organizations in our industry?
- What cloud‑focused attack techniques are trending right now?
- Which vulnerabilities are most likely to be exploited against us?

These questions can/should be driven by concerns such as business risks, threat model gaps, and detection/response priorities.

**2. Collection**  
Once a question has been defined, data must be gathered from both internal and external sources.

Internal sources:  
- SIEM and EDR telemetry
- Incident response reports
- Threat hunting findings

External sources:  
- OSINT
- Commercial intel feeds
- Vendor reports and advisories
- Government and ISAC sharing

**3. Processing**  
In this stage of the lifecycle analysts need to 'clean' the data by normalizing, enriching, and structuring. For example:
- Parsing indicators into structured formats
- Enriching IPs and domains with ASN, geolocation, or reputation
- Mapping observed activity to MITRE ATT&CK

**4. Analysis**  
Turn processed data into a "story." This is where analysts identify patterns and provide context, answering questions such as:  
- What happened?
- Why does it matter?
- How does this affect us specifically?

**5. Dissemination**  
Of course, none of this really matters until the right information is passed along to the right people. Depending on the content, intelligence reports need to be disseminated to the appropriate audience:  
- Executive briefs
- SOC-ready detection recommendations
- Threat hunting leads
- Incident response playbooks

**6. Feedback and Refinement**  
Reporting should be reviewed for usefulness, relevance, and impact, and then improved upon for the next cycle. Some key considerations include:  
- Did this intel change a decision?
- Did it improve detections or response?
- Was it timely and relevant?

{{< css.inline >}}
<style>
.canon { background: white; width: 100%; height: auto; }
</style>
{{< /css.inline >}}
