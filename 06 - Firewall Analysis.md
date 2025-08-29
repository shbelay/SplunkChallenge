# Day 6: Palo Alto Firewall Log Analysis

## Overview
On this day, we reviewed Palo Alto-generated firewall logs (`panos_threat_log.json`) to explore how threat logs are recorded, analyzed, and used for detection. The goal was to understand how different fields contribute to event classification and response, and to answer three practical analysis questions.

---

## What is a Palo Alto Threat Log?
Palo Alto threat logs are generated when firewall traffic matches a **security profile**, such as:
- Anti-Spyware
- Vulnerability Protection
- Antivirus
- WildFire
- DNS Security

Each entry in the log typically includes:
- **src/dst IPs & ports**: The source and destination addresses and ports
- **app/protocol**: Detected application or protocol (e.g., `ssl`, `smb`, `rdp`)
- **rule**: The specific firewall rule that allowed or blocked the traffic
- **threat_name & subtype**: Threat signature or behavioral category (e.g., `C2 Callback`, `Malware.Downloader`)
- **severity**: Risk level of the detected event (e.g., informational → critical)
- **action**: What the firewall did in response (e.g., blocked, allowed, alerted, reset)

---

## Questions We Answered
### Question #1
**Which source IPs have the highest count of `action=blocked` events?**
> This analysis helps identify the most aggressive or potentially malicious sources being actively blocked by the firewall.

### Question #2
**What apps/ports are most commonly targeted in blocked events?**
> Useful for identifying which services (e.g., `rdp`, `smb`, `ssl`) are being attacked and adjusting firewall policies or segmentation accordingly.

### Question #3
**Which severity level is most common for events triggered by the `Block-C2` rule?**
> Helps gauge the risk level of Command-and-Control related detections and understand how often they occur at different severity tiers.

---

## Summary
Palo Alto threat logs provide rich context for security analysis. Reviewing these logs allows defenders to:
- Identify top attacking sources
- Prioritize services under threat
- Track threat behavior using rules and severity levels

These logs are essential for building threat detection dashboards and automating alerts in SIEM platforms like Splunk, Elastic, or XDR tools.
