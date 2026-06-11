# SOC Homelab — Phishing Credential Harvesting Simulation

## Project Overview
A hands-on SOC homelab simulating a phishing credential harvesting 
attack using the Social Engineering Toolkit (SET) on Kali Linux, 
with detection and alerting via Splunk SIEM.

## Lab Environment
| Component | Details |
|---|---|
| Attacker | Kali Linux 2025.2 (VirtualBox) — 10.0.2.3 |
| Victim | Ubuntu 22.04 (VirtualBox) — 10.0.2.15 |
| SIEM | Splunk Enterprise 10.2.3 (Windows host) |
| Network | NAT — 10.0.2.0/24 |

## Tools Used
- Social Engineering Toolkit (SET) — phishing page cloning
- tcpdump — network packet capture on victim
- Splunk Universal Forwarder — log shipping from Ubuntu
- Splunk Enterprise — detection, alerting, dashboards

## Attack Walkthrough

### Phase 1 — Setup (Attacker)
Launched SET on Kali and selected:
- Social Engineering Attacks
- Website Attack Vectors
- Credential Harvester Attack Method
- Site Cloner → https://github.com/login

SET cloned the GitHub login page and started
a credential harvester on port 80.

### Phase 2 — Delivery (Victim)
Victim VM opened Firefox and navigated to:
http://10.0.2.3

The cloned GitHub login page was served.
Victim submitted fake credentials:
- Username: testattempt@gmail.com
- Password: fakepassword123

### Phase 3 — Harvest (Attacker)
SET terminal displayed:
POSSIBLE USERNAME FIELD FOUND: login=testattempt@gmail.com
POSSIBLE PASSWORD FIELD FOUND: password=fakepassword123

### Phase 4 — Detection (SOC Analyst)
tcpdump ran on Ubuntu during the attack:
sudo tcpdump -i enp0s3 host 10.0.2.3 -A -w /tmp/phishing.pcap

Capture converted and forwarded to Splunk:
index=main sourcetype=phishing_capture "POST"
→ 2 events showing HTTP POST /session HTTP/1.1
  from ubuntu.50946 > 10.0.2.3.http

## Splunk Detection Query
index=main sourcetype=phishing_capture "POST"

## Splunk Alert
- Name: Phishing Credentials Harvest Detected
- Trigger: Number of results > 0
- Severity: Critical
- Type: Real-time

## MITRE ATT&CK Mapping
| ID | Technique | Description |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | Victim directed to cloned login page |
| T1056.003 | Input Capture: Web Portal Capture | Credentials harvested via fake form |
| T1071.001 | App Layer Protocol: Web Protocols | HTTP POST exfiltrated credentials |

## Key Findings
1. SET successfully cloned GitHub login and harvested credentials
2. tcpdump captured the full HTTP POST including username and password
3. Splunk ingested network capture and detected credential submission
4. Real-time alert created for future phishing detection

## Detection Gap Identified
Standard syslog does NOT capture browser HTTP traffic.
Network-layer capture (tcpdump, Zeek, or a proxy) is required
to detect phishing credential submissions — a realistic SOC
limitation that reinforces the need for layered detection.

## Screenshots
- SET credential harvest output (Kali terminal)
- tcpdump HTTP POST capture (Ubuntu terminal)
- Splunk events showing phishing_capture sourcetype
- Splunk alert configuration

## Related Project
[SOC Homelab — SSH Brute-Force Detection]
(https://github.com/Juzrajesh/soc-homelab-ssh-bruteforce)
