**Task 3: Victim Profile & Threat Mapping**

*Cyber Threat Intelligence Sprint — Desert Oil*

## Methodology

Three victims were identified in OpenCTI using AlienVault OTX-sourced reporting within the last 3 months (June 28 - September 28, 2026), prioritizing the most recently published reports. Victims were selected to reflect the platform's top-ranked "Most Targeted Victims" categories over the trailing 90 days: Government, Technology, and Finance. Each report was reviewed via its Content and Entities tabs in OpenCTI to extract the threat actor, malware/tooling, infrastructure indicators, and MITRE ATT&CK techniques used to build the Diamond Model and Kill Chain breakdowns below. Supporting screenshots from OpenCTI are placed at the end of each victim's section.

# Victim 1: "Mind the (Patch) Gap, Part 2" — Chrome & Windows 0-Day Exploit Chain Against Government Entities

**Victim: Asian government entities (Government sector; also tagged NGOs, Media)**

## Top Threat

UTA0565 (Chinese threat actor)

## Actor / Campaign Overview

In early September 2026, Chinese threat actor UTA0565 exploited unpatched zero-day vulnerabilities in Google Chrome and Microsoft Windows through sophisticated phishing campaigns.

The actor registered fake domains impersonating legitimate organizations, including China Digital Times and the Center for American Progress, and sent targeted phishing emails to Asian government entities. Victims were directed to spoofed websites hosting an exploit chain leveraging CVE-2026-85046, CVE-2026-87491, and CVE-2026-85880.

The attacks delivered CLEANGULP, a previously undocumented malware family with backdoor capabilities including command execution, file operations, and beacon object file execution. The malware communicated with command-and-control infrastructure via encrypted HTTP traffic using custom encoding. Multiple Chinese APT groups demonstrated coordinated access to this exploit kit, suggesting widespread sharing within the Chinese cyber espionage community during the vulnerability window.

## Diamond Model

| **Vertex** | **Detail** |
| --- | --- |
| Adversary | UTA0565 — Chinese threat actor; exploit kit access shared across multiple Chinese APT groups during the vulnerability window. |
| Capability | CLEANGULP backdoor (command execution, file operations, beacon object file execution); zero-day exploit chain across CVE-2026-85046, CVE-2026-87491, and CVE-2026-85880 (Chrome and Windows); spear phishing with spoofed impersonation domains. |
| Infrastructure | Spoofed domains impersonating China Digital Times (chinadigitaltimes.top) and the Center for American Progress (americanprgoress.top); additional infrastructure including thecovnresation.com/.net, personclouds.com, halal-navi.net, halaltak.net, outsourcingwise.net; IP 96.9.125.52; C2 via encrypted HTTP with custom encoding. |
| Victim | Asian government entities. |

## Timeline

-   Early September 2026: Campaign active — phishing emails delivered, exploit chain deployed against victims via spoofed websites.
-   September 23, 2026: Publicly reported (most recent incident selected for this report).

## Kill Chain Stage(s)

-   Initial Access — T1566.002 (Spearphishing Link), T1189 (Drive-by Compromise), T1204.001 (Malicious Link)
-   Execution — T1203 (Exploitation for Client Execution), T1059.003 (Windows Command Shell)
-   Persistence — T1547.001 (Registry Run Keys), T1053.005 (Scheduled Task)
-   Privilege Escalation — T1068 (Exploitation for Privilege Escalation)
-   Defense Evasion — T1036.005 (Masquerading), T1140 (Deobfuscate/Decode Files), T1027.002 (Software Packing)
-   Discovery — T1082 (System Information Discovery), T1033 (System Owner/User Discovery), T1057 (Process Discovery)
-   Command and Control — T1071.001 (Web Protocols), T1573.001 (Symmetric Cryptography), T1571 (Non-Standard Port), T1132.001 (Standard Encoding), T1105 (Ingress Tool Transfer)
-   Exfiltration — T1041 (Exfiltration Over C2 Channel)

## Analyst Note

Multiple Chinese APT groups were observed with coordinated access to the same 0-day exploit kit during the vulnerability window, suggesting shared tooling or infrastructure within the Chinese cyber-espionage ecosystem — a useful talking point on the broader threat landscape for the executive briefing.

## Supporting Screenshots

*![](./Task%203_Report_images/image-001.png)Report Overview page — "Mind the (Patch) Gap, Part 2"*

*![](./Task%203_Report_images/image-002.png)Entities tab — UTA0565, CLEANGULP, CVEs, Attack Patterns*

*![](./Task%203_Report_images/image-003.png)*

*Observables tab — spoofed domains and IP indicators*

# Victim 2: "Open Season on Kapibala" Government Records Breach via WordPress/ZyXEL Exploitation

**Victim: A western government organization (primary breach); 996 ZyXEL devices compromised across 48 countries (secondary mass-exploitation campaign)**

## Top Threat

Suspected Chinese-speaking threat actor, linked to Red Heron.

## Actor / Campaign Overview

A suspected Chinese-speaking threat actor, operating from a UTC+8 timezone and linked to Red Heron, conducted extensive exploitation campaigns from May through September 2026.

The adversary maintained persistent use of a single IP address to target multiple technologies, including WordPress, ZyXEL switches, and various enterprise platforms. In the most severe incident, the attacker exploited WordPress vulnerabilities CVE-2026-63030 and CVE-2026-60137 to breach a western government organization, stealing over 18,000 sensitive records containing credentials and personally identifiable information.

The actor also exploited CVE-2026-7273 in ZyXEL GS1900 switches, compromising 996 devices across 48 countries and exfiltrating configurations and credentials. The attacker demonstrated sophisticated capabilities including custom tool development (possibly leveraging large language models), privilege escalation attempts, and systematic data exfiltration using PowerShell-based collection and staging techniques. OpenCTI confidence level: Confirmed by Other Sources.

## Diamond Model

| **Vertex** | **Detail** |
| --- | --- |
| Adversary | Suspected Chinese-speaking actor, UTC+8 timezone, linked to Red Heron intrusion set. |
| Capability | Custom tooling (possibly LLM-assisted development); WordPress exploitation; ZyXEL GS1900 switch exploitation; PowerShell-based collection/staging; privilege escalation. |
| Infrastructure | Single persistent IP address reused across the campaign (e.g. 74.48.66.73, 172.245.247.21, 104.225.153.141); labeled "chinese-speaking actor" across 18+ associated indicators. |
| Victim | Western government organization (18,000+ records: credentials and PII exfiltrated); 996 ZyXEL devices across 48 countries (configs and credentials exfiltrated). |

## Timeline

-   May - September 2026: Sustained exploitation campaign.
-   Vulnerabilities exploited: CVE-2026-63030 and CVE-2026-60137 (WordPress); CVE-2026-7273 (ZyXEL GS1900).
-   Publicly reported: September 22, 2026.

## Kill Chain Stage(s)

-   Initial Access — T1190 (Exploit Public-Facing Application)
-   Execution — T1059.001 (PowerShell)
-   Persistence — T1505.003 (Web Shell), T1136.001 (Local Account), T1078 (Valid Accounts)
-   Privilege Escalation — T1548.002 (Bypass User Account Control)
-   Credential Access — T1003.001 (LSASS Memory), T1003.002 (Security Account Manager), T1552.001 (Credentials in Files), T1555 (Credentials from Password Stores), T1110.003 (Password Spraying)
-   Discovery — T1087.001 (Local Account), T1083 (File and Directory Discovery), T1082 (System Info Discovery), T1033 (System Owner Discovery)
-   Lateral Movement — T1021.002 (SMB/Windows Admin Shares)
-   Collection — T1005 (Data from Local System), T1560.001 (Archive via Utility)
-   Command and Control — T1105 (Ingress Tool Transfer)
-   Exfiltration — T1567.002 (Exfiltration to Cloud Storage)

## Analyst Note

This incident maps to 10 of the 14 Global Kill Chain stages with named ATT&CK technique IDs, making it the most comprehensively documented incident of the three victims profiled in this report.

## Supporting Screenshots

*![](./Task%203_Report_images/image-004.png)*

*Report Overview page — "Open Season on Kapibala" (description, radar chart, labels/CVEs)*

 *![](./Task%203_Report_images/image-005.png) Entities tab — Red Heron, malware, CVEs, Attack Patterns*

*![](./Task%203_Report_images/image-006.png)*

*Observables tab — IP/indicator list*

# Victim 3: "RemControl" — Android Banking Trojan Malware-as-a-Service Campaign

**Victim: 30+ financial institutions across Western Europe, the Middle East, and Canada**

## Top Threat

Threat actor UNKK (RemControl Android banking trojan, Malware-as-a-Service)

## Actor / Campaign Overview

Group-IB researchers discovered RemControl, a previously undocumented Android banking trojan operating as Malware-as-a-Service since May 2026.

The malware targets over 30 financial institutions across Western Europe, the Middle East, and Canada through fake TVTap IPTV download pages distributed via malvertising. RemControl abuses the Android Accessibility Service to inject phishing overlays, stream device screens in real time, log keystrokes, and provide full remote control.

The investigation revealed that the criminal infrastructure was inadvertently built using AI assistance, with complete AI-generated responses found verbatim in production phishing pages; the C2 panel documentation describes credential theft as "quiz completion," suggesting the developers deceived AI assistants into building fraud infrastructure under false pretenses.

The threat actor UNKK operates the campaigns, with a possible link to the Medusa UNKN affiliate botnet based on overlapping infrastructure patterns and naming conventions.

## Diamond Model

| **Vertex** | **Detail** |
| --- | --- |
| Adversary | Threat actor UNKK; possible affiliate link to the Medusa UNKN botnet. |
| Capability | RemControl (Android banking trojan, Malware-as-a-Service); Accessibility Service abuse for overlay injection, screen streaming, keylogging, and remote control. |
| Infrastructure | Fake TV Tap IPTV distribution page (tvtap-liveapp.com); C2/VPN infrastructure (vpn.doneplay.site, vpn.askarzadeh.com); additional domains (definatelynoone.com, bnbnhura.top, cdn.dlmafi.top); malvertising delivery chain. |
| Victim | 30+ financial institutions across Western Europe, the Middle East, and Canada. |

## Timeline

-   Since May 2026: RemControl operating as Malware-as-a-Service.
-   Publicly reported: September 23, 2026.

## Kill Chain Stage(s)

-   Resource Development — building/acquiring MaaS infrastructure, reportedly using AI-assisted tooling to construct phishing and C2 components
-   Initial Access — malvertising leading to fake IPTV application downloads
-   Execution / Persistence — Android Accessibility Service abuse for persistent overlay control
-   Collection — real-time screen streaming, keystroke logging
-   Command and Control — dedicated C2 panel, VPN-fronted infrastructure
-   Impact — credential/PIN theft enabling account takeover and financial fraud

## Analyst Note

Notable for the sprint's executive briefing: the criminal developers appear to have unknowingly used AI assistance to help build phishing and fraud infrastructure, obscuring their true purpose from the AI tool. This is a strong talking point for the board presentation on emerging AI-enabled threat trends.

*![](./Task%203_Report_images/image-007.png)*

*Report Overview page — "RemControl: AI Built the Overlays. Victims Lose their PINs" (description, labels)*

*![](./Task%203_Report_images/image-008.png)*

*Entities tab — UNKK, RemControl, related malware/countries*

*![](./Task%203_Report_images/image-009.png)*

*Observables tab — domains and VPN indicators.*

# Summary

The three victims profiled above were selected to represent the platform's top three victim categories over the trailing 90 days (Government, Technology, and Finance), prioritizing the most recently published incidents available in OpenCTI (all three reports were published between September 22-23, 2026).  
  
Together they illustrate three distinct threat patterns relevant to Desert Oil's risk posture: coordinated nation-state exploitation of browser/OS zero-days against government targets (Victim 1), large-scale opportunistic exploitation of public-facing and network infrastructure (Victim 2), and AI-enabled financial fraud tooling (Victim 3).