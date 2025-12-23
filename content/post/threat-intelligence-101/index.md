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

Now that I've covered the basics of threat hunting, let's pivot to threat intelligence. The intent of this writeup is to provide a simple, broad overview of threat intelligence, and to touch on how threat intelligence integrates with threat hunting.

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

## Pillars of Threat Intelligence

Not all intelligence is created equally. To be effective, intel must be tailored for both audience and focus. There are four primary "pillars" of intelligence that we will discuss. The distinction between each of these four types is the difference between a reactive SOC and a proactive security program.

### Strategic Intelligence
- Target Audience:
  - CISOs, Board Members, Executives, Risk Leaders
- Focus:
  - High-level trends, risk landscape, and geopolitical shifts
- Example: 
  - A report on how AI-driven phishing is specifically targeting the financial sector's regional banks

### Operational Intelligence
- Target Audience:
  - SOC Managers, IR Leaders
- Focus:
  - Details on specific campaigns, intrusion sets, and attacker workflows
- Example: 
  - A walkthrough of a specific ransomware campaign from initial access to data exfiltration

### Tactical Intelligence
- Target Audience:
  - CISOs, SOC Analysts, Detection Engineers, Threat Hunters
- Focus:
  - TTPs, MITRE ATT&CK mappings - the mechanics of an attack
- Example: 
  - Adversaries abusing OAuth apps for persistence in cloud tenants

### Technical Intelligence
- Target Audience:
  - SIEMs, EDR Systems, other SOC Tools and Automation
- Focus:
  - IOCs like IPs, hashes and domains - machine-readable data
- Example: 
  - A list of 500 malicious IP addresses associated with a current ransomware campaign.

## Sources of Threat Intelligence

### Open-Source Intelligence (OSINT)
OSINT is the process of collecting and analyzing information that is publicly available. It is the "free" foundation of most CTI programs.  

**Government & Regulatory Feeds**   
Reports from agencies like CISA (Known Exploited Vulnerabilities catalog) or FBI InfraGard.

**Security Blogs & Research**  
Deep dives from companies like Mandiant, SentinelOne, or independent researchers on platforms like X (Twitter) or Mastodon.

**Technical Repositories**  
Scanning tools like Shodan (for internet-connected devices) or VirusTotal (for checking if a file is malicious).

**Vulnerability Databases**  
The CVE (Common Vulnerabilities and Exposures) list tells you which software bugs are currently being exploited in the wild.

### Commercial Intelligence Vendors
If OSINT is the local news, Commercial Intelligence is a private investigative firm. Organizations pay for these services to get high-fidelity data that isn't available to the general public.

**Proprietary Telemetry**  
Vendors like CrowdStrike or Microsoft see what is happening across millions of global endpoints. They turn this massive data set into "early warnings" for their customers.

**Adversary Dossiers**  
Deep-dive profiles on specific hacker groups. Instead of just "an IP address," you get a report on "Fancy Bear" (a Russian state actor), including their typical office hours, preferred malware, and historical targets.

**Finished Intelligence**  
Human-curated reports that explain the "So What?" for executives. This saves your team hours of manual research by providing "ready-to-consume" analysis.

**Data Breach & Dark Web Monitoring**  
Many vendors (like Recorded Future or ZeroFox) specifically crawl illicit marketplaces to see if your company's leaked credentials or "stolen" data are being sold before a breach becomes public.

### Information Sharing Communities
This sits between public and private intelligence and often reqeuires more specialized access.

**ISACs (Information Sharing and Analysis Centers)**  
Industry-specific groups (e.g., FS-ISAC for Finance, H-ISAC for Healthcare) where competitors share threat data to protect the whole sector.

**Dark Web Monitoring**  
Specialized tools that "scrape" illicit forums and marketplaces to see if your company's credentials or "leaked" data are being sold.

**SOCMINT (Social Media Intelligence)**  
Monitoring social trends to spot coordinated "hacktivist" campaigns or early mentions of a new zero-day exploit.

### Internal Intelligence
Internal intelligence is often the most valuable because it is 100% relevant to your specific environment. 

**Endpoint Telemetry (EDR/XDR)**  
Your local "black boxes." These provide details on process executions, registry changes, and lateral movement attempts.

**Network & Flow Logs**  
Firewall, DNS, and Proxy logs. These reveal where your data is going and if your machines are "calling home" to a Command & Control (C2) server.

**Incident Response (IR) Reports**  
Looking at past "post-mortems" from your own company. Attackers often return to the same victim using the same methods if they weren't fully evicted.

**Internal Honeypots**  
Decoy systems like "fake" file servers or unpatched databases. Anyone touching these is automatically a high-fidelity threat.

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

## Threat Intelligence vs Threat Hunting
Threat intelligence and threat hunting are mutually reinforcing.

Threat Intelligence enables Hunting by:
- Identifying relevant adversaries
- Highlighting emerging techniques
- Providing hypotheses to test

Threat Hunting feeds Intelligence by:
- Validating or disproving reports
- Discovering novel activity
- Adding environment‑specific context

## Operationalizing Threat Intelligence
Like with threat hunting, threat intelligence needs to result in actionable improvements to be useful, impactful, and successful. With the appropriate audience and focus, threat intel can drastically improve many processes across a security team.

Detection Engineering
- Translate TTPs into SIEM and EDR detections
- Focus on behaviors, not just indicators

Threat Hunting
- Build hunts around adversary tradecraft
- Prioritize based on likelihood and impact

Incident Response
- Accelerate scoping and containment
- Improve attribution and confidence

Vulnerability Management
- Prioritize patching based on active exploitation
- Align exposure with threat actor capability

## Final Thoughts
Threat intelligence is only valuable when it drives action. The goal is not to know everything about the threat landscape, but to know the right things that help your organization detect faster, respond smarter, and reduce risk. When threat intelligence is tightly integrated with threat hunting, detection engineering, and incident response, it becomes a force multiplier rather than a reporting function.

{{< css.inline >}}
<style>
.canon { background: white; width: 100%; height: auto; }
</style>
{{< /css.inline >}}
