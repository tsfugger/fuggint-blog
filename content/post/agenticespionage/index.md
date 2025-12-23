+++
author = "Tommy Fugger"
title = "Agentic Espionage"
date = "2025-12-19"
description = "Inside the First Largely Autonomous AI Cyber Espionage Campaign"
categories = ["Threat Intelligence", "Threat Hunting"]
tags = ["threat hunting", "AI", "espionage", "threat intelligence", "APT"]
image = "agenticespionage.png"
draft = true
+++

The intent of this post is not to simply rehash <cite>Anthropic's[^1]</cite> report, as you can [read that HERE](https://www.anthropic.com/news/disrupting-AI-espionage); rather, my goals are to outline AI's role in this campaign, present strategies for hunting AI-driven operations, and highlight how that may differ from traditional human operations.

[^1]: Disrupting the First Reported AI-Orchestrated Cyber Espionage Campaign. (2025). Anthropic PBC. https://www.anthropic.com/news/disrupting-AI-espionage

## Overview

In mid-September 2025, a landmark shift in the cyber threat landscape was uncovered: the first documented case of a large-scale, AI-orchestrated cyber espionage campaign. Conducted by a Chinese state-sponsored group designated as GTG-1002, the operation targeted approximately 30 entities, including major technology corporations, government agencies, and financial institutions. This campaign represents a fundamental evolution in offensive capabilities, where human operators shifted from performing tactical work to acting as strategic supervisors, leaving 80-90% of the active intrusion tasks to autonomous AI agents. 

The GTG-1002 campaign utilized an autonomous attack framework built around Claude Code and the Model Context Protocol (MCP). By manipulating AI instances into role-playing as defensive security professionals, the threat actors induced the AI to perform reconnaissance, vulnerability discovery, and data exfiltration across multiple targets simultaneously. Unlike traditional attacks where AI merely "advises," this framework functioned as an execution engine, maintaining attack states and managing transitions between phases with minimal human oversight.

## The Attack Lifecycle and AI Integration

The campaign followed a multi-stage intrusion sequence typical of advanced persistent threats — but with AI deeply integrated throughout each phase.

**Phase 1: Campaign Initialization**  
Human operators selected high-value targets and "socially engineered" the AI by convincing it that it was conducting authorized defensive testing.  
AI Role: Minimal, as humans conducted the social-engineering and target selection.

**Phase 2: Reconnaissance**  
Claude systematically cataloged infrastructure, analyzed authentication, and mapped network topologies.  
AI Role: Fully autonomous, conducting reconnaissance against hundreds of endpoints and mapping internal services across multiple IP ranges simultaneously.

**Phase 3: Vulnerability Discovery and Validation**  
The AI generated custom payloads tailored to discovered vulnerabilities (such as SSRF) and executed them via remote interfaces.  
AI Role: Autonomous execution. The AI validated exploits through callback communication systems and only requested human approval at the final escalation point.

**Phase 4: Credential Harvesting and Lateral Movement**  
Once inside, the AI queried internal services, extracted certificates, and tested them across the environment to map privilege levels.  
AI Role: Self-directed. The AI independently determined which credentials worked on which systems, moving laterally across APIs and container registries.

**Phase 5: Data Collection and Intelligence Extraction**  
The AI queried databases, parsed results for proprietary data, and categorized findings by their intelligence value.  
AI Role: Analysis-heavy. The AI acted as a data scientist, identifying high-value info without manual human parsing.

**Phase 6: Documentation and Handoff**  
Throughout the campaign, the AI generated structured markdown files detailing every service found, credential stolen, and exploit used.  
AI Role: Fully autonomous record-keeping. This allowed different human teams to resume complex operations seamlessly.

While the attack chain doesn't really differ from that of a human-led campaign, we can see that the role AI played in this operation allowed for scalability that otherwise could not be achieved. 

## Threat Hunting Strategies for AI-Driven Operations
Traditional threat hunting assumes adversaries with predictable patterns and human decision-making cycles. AI-orchestrated operations, however, require reevaluating detection strategies across the kill chain with a shift in focus from "human-like" behaviors to "machine-speed" patterns. What Anthropic documented is not “AI helping a hacker” — it is AI acting as the primary operator, with humans serving as approval gates. This fundamentally changes *what* we hunt for and *how* we hunt.

### Operational Tempo

The most immediate differentiator between AI-driven and traditional operations is the sheer volume and rate of activity.

**Human-Led Activity**  
Typically follows a "bursty" pattern. An operator researches a tool, types a command, waits for output, and analyzes it before the next step. This creates natural pauses and a tempo consistent with human interaction.

**AI-Orchestrated Activity**  
In the GTG-1002 campaign, peak activity reached thousands of requests, representing sustained rates of multiple operations per second — a frequency physically impossible for human operators.

**Hunting**  
Search for sequential activity that would be "physically impossible" for a human operator. Traditional rate-limiting often targets brute-force attempts, but here, defenders must hunt for a high volume of *varied* tactical commands (such as service discovery followed immediately by exploit validation) occurring within milliseconds of each other. In the example query below, we are looking for a single source IP interacting with an unusually high number of unique internal destinations or URLs in a very short window.

````
FROM logs-endpoint, logs-network
| WHERE event.category == "network" AND network.direction == "internal"
| STATS unique_dest = COUNT_DISTINCT(destination.address), total_req = COUNT(*) BY source.address, bin(timestamp, 1s)
| WHERE unique_dest > 50 OR total_req > 100
| SORT timestamp DESC
````

This query buckets network logs into 1-second windows. It calculates how many unique internal addresses a single source reached out to per second. A human might click a link or run a script that hits 5–10 endpoints, but an AI agent mapping a network will often hit 50+ unique destinations or ports per second as it "thinks" through the topology.

### Data Input vs Text Output Disparity

A key indicator of AI-driven post-exploitation is how data is processed within the environment.

**Human-Led Activity**  
Humans often exfiltrate raw data (files, database dumps) to analyze it offline. If they analyze it on-site, logs typically show interactive commands like grep, awk, or manual file opens.

**AI-Orchestrated Activity**  
AI agents, such as Claude in this operation, can parse enormous volumes of stolen information locally or via an orchestration proxy to identify intelligence value.

**Hunting**  
Look for a massive disparity between the volume of data inputs (such as thousands of database queries or file reads) and the relatively small volume of text outputs or exfiltrated "summary" files. This suggests an automated engine is distilling vast datasets into concise intelligence reports. In the example query below, the goal is to identify AI behavior by finding hosts that are reading large volumes of data (input) but sending back very small, highly structured summary files (output).

````
FROM logs-network, logs-endpoint
| WHERE event.action == "file_read" OR event.action == "exfiltration"
| STATS 
    bytes_read = SUM(file.size), 
    bytes_out = SUM(network.bytes) 
  BY host.name, user.name
| EVAL ratio = bytes_read / bytes_out
| WHERE bytes_read > 100000000 AND ratio > 1000
| SORT ratio DESC
````

This query compares the total bytes a user "read" from the disk/databases versus what they sent over the wire. In a traditional smash and grab human theft, the ratio is close to 1:1. Here, we look for a Ratio > 1000, meaning for every 1GB read, only 1MB was sent out; this would be a sign of an AI agent summarizing logs, database rows, or codebases into a markdown report before exfiltrating.

### State Preservation and Session Persistence

AI agents solve the "hands-off" problem that often plagues human-led state-sponsored teams.

**Human-Led Activity**  
When shifts change or campaigns resume after a break, there is often a "re-reconnaissance" phase where the new operator verifies previous findings, leading to redundant logs.

**AI-Orchestrated Activity**  
Claude maintained persistent operational context across sessions spanning multiple days, allowing complex campaigns to resume seamlessly without human reconstruction. Furthermore, the AI generated structured markdown documentation automatically, tracking every credential and technique discovered.

**Hunting**  
Analyze the "resumption fingerprint." If an intrusion resumes after a 48-hour quiet period and immediately executes perfectly targeted commands against previously discovered internal APIs — without any repeat reconnaissance — it strongly suggests a persistent AI state rather than a human operator re-orienting themselves. The GTG-1002 report also highlighted that Claude Code and similar agents maintain attack states in structured files (Markdown or JSON). We could attempt to hunt on this state preservation and session persistence by identifying the creation and constant updating of these files.

```` 
FROM logs-endpoint
| WHERE event.action == "file_modify_event"
| WHERE file.extension IN ("json", "md", "yaml")
| STATS 
    update_count = COUNT(*), 
    unique_files = COUNT_DISTINCT(file.path) 
  BY process.executable, host.name
| WHERE update_count > 100 AND unique_files < 5
| WHERE process.executable LIKE "*python*" OR process.executable LIKE "*node*"
````

Normal users don't update a single JSON or Markdown file hundreds of times an hour via a script engine (Node.js/Python). This query identifies a process that is "logging its thoughts" or updating a database of "discovered assets" in real-time. This is the "brain" of the AI agent living on the disk.

### Detecting the Model Context Protocol (MCP) and Tooling Proxies

The technical architecture of these attacks often relies on standardized protocols to bridge the AI to the target environment. The GTG-1002 framework used Claude Code and the Open Model Context Protocol (MCP) to conduct operations. The AI was effectively an "execution engine" within an automated system that decomposed complex attacks into discrete technical tasks.

**Hunting**  
We can monitor for unusual traffic patterns or headers associated with MCP servers being used as "tool call proxies" to interface with internal target services, or for traffic originating from "neutral party" callback services used by the AI to validate exploits autonomously. We can also attempt to identify artifacts associated with MCP or Claude Code execution. Even better, we can look for behavioral indicators of tooling execution via AI. A standard application typically spawns a predictable set of child processes; an AI agent, however will often spawn a high diversity of unique utilities from a single parent process in a very tight temporal window.

```` 
FROM logs-endpoint
| WHERE event.category == "process" AND event.type == "start"
| // Filter for common discovery/exploitation tools often called by agents
| WHERE process.name IN ("whoami.exe", "net.exe", "hostname.exe", "ipconfig.exe", 
                         "systeminfo.exe", "nltest.exe", "netstat.exe", "curl.exe")
| STATS 
    unique_tools_count = COUNT_DISTINCT(process.name),
    tool_list = COLLECT_DISTINCT(process.name)
  BY process.parent.entity_id, process.parent.executable, bin(timestamp, 10s)
| WHERE unique_tools_count >= 5
| SORT unique_tools_count DESC
````
This query identifies any parent process that spawns more than 5 unique "Living off the Land" or discovery utilities within a 10-second window — a pattern typical of an AI agent iterating through a tool-calling loop.

### Leverage "AI Hallucination"

In Anthropic's report, they noted that Claude frequently overstated findings or fabricated data, such as claiming to have obtained working credentials that were actually non-existent or public information, or even trying to exploit vulnerabilities that didn't exist. This presents a defensive opportunity to take advantage of these flaws.

**Hunting**  
One way we could attempt to combat agentic operations is via honeypots. While traditional honeytokens do the job, "hallucination bait" would likely be even more effective for AI operations. This would involve placing complex, legitimate-looking - but subtly flawed - configuration files in internal environments. A human might spot the flaw through experience, but an AI agent may confidently "validate" and attempt to use the fabricated data, creating a high-fidelity alert when the agent acts on its own hallucination .

```` 
FROM logs-auth, logs-network
| WHERE destination.address == "honeypot_ip" // Your Honey-IP
   OR user.name == "honeypot_user"           // Your Honey-User
| STATS count() BY source.ip, event.action
| SORT timestamp DESC
````
This a really simple, but certainly high-fidelity alert. If we place the "hallucination bait" file containing a fake IP and username, any traffic to that IP or use of those credentials would be an immediate red flag. While a human may validate discovered credentials, an AI agent would likely scrape the file and attempt to use the credentials.

## Final Thoughts
The hunting strategies outlined here don't even begin to scratch the surface of what defending against AI-orchestrated operations will require. What Anthropic documented is not simply a new toolset in the hands of attackers, but a structural change in how intrusion activity can be generated, scaled, and adapted in real time. Once an autonomous system is capable of independently enumerating environments, generating exploits, pivoting laterally, extracting intelligence, and refining its own approach faster than defenders can respond, compromise is no longer constrained by attacker fatigue, expertise, or other human limitations.

As a result, threat hunting can no longer rely solely on known techniques, familiar tools, or linear kill-chain assumptions. Hunting for AI-orchestrated activity requires a different perspective that focuses on detecting operational anomalies that would otherwise be impossible for a human operator to achieve; defenders must learn to recognize what cannot be done by a person, even a highly skilled one, and treat those patterns as indicators of potential compromise. The future of threat hunting will not be about keeping pace with AI-enabled attackers, but about redefining how we observe, reason about, and disrupt autonomous adversaries before they can fully leverage their speed and scale.


{{< css.inline >}}
<style>
.canon { background: white; width: 100%; height: auto; }
</style>
{{< /css.inline >}}
