Cyber Threat Intelligence Sprint

**Task 2: National Threat Landscape**

Organization: Desert Oil| Team 6 | Headquarters: Saudi Arabia

Platform: OpenCTI (AlienVault OTX connector) | Date: September 27, 2026

# Top 2 Threats Targeting Saudi Arabia

Using OpenCTI's Saudi Arabia country entity as the anchor point, ten entities were found with a “targets” relationship to the country: three Intrusion Sets (Tortoiseshell, DragonForce, LinX Coders) and seven Malware/tooling entities (AntSword, GoldKefu, GoldDigger, Downloader Module, Module Installer, BADBOX, and a second DragonForce entry). Candidates were narrowed to actor-level Intrusion Sets whose reporting explicitly discusses Saudi Arabia. LinX Coders was excluded (its sole report is a US-centric Microsoft 365 phishing operation with no mention of Saudi Arabia) and BADBOX was excluded (a global Android/IoT fraud botnet with no country-specific reporting).

The top 2 threats currently targeting Saudi Arabia are:

-   **1\. DragonForce:** a financially motivated ransomware group with a report titled around targeting Saudi Arabia and the deepest intelligence footprint in OpenCTI.
-   **2\. Tortoiseshell:** An Iranian-linked cyber-espionage group whose most recent reporting (Aug 2026) names Saudi Arabia among its targeted countries.

| **Threat** | **Reports** | **Indicators** | **Malware / Tools** | **ATT&CK Techniques** | **Vulnerabilities** |
| --- | --- | --- | --- | --- | --- |
| DragonForce | 3 | 69 | 4 | 37 | 12 |
| Tortoiseshell | 1 | 2 | 1 | 15 | 0 |

| **![](./Task%202_Report_images/image-001.png)**   **![](./Task%202_Report_images/image-002.png)** |
| --- |

*Figure 1: OpenCTI Saudi Arabia country entity and the threats linked to it*

# Threat Descriptions

## 1\. DragonForce

**Classification:** Intrusion Set — financially motivated ransomware operation

**Recorded targeting:** Saudi Arabia and the United States; construction sector

**Behavior**

DragonForce is a ransomware operation with an evolving intrusion chain. Recent reporting ties it to exploitation of CVE-2025-5777 (“CitrixBleed 2”) on Citrix NetScaler for initial access, abuse of Microsoft Teams relay infrastructure to hide command-and-control traffic through a backdoor (Backdoor.Turn), and bring-your-own-vulnerable-driver (BYOVD) techniques to impair security tools. After entry it relies on credential theft and remote-execution tooling, collects and archives data, exfiltrates it, and deploys ransomware for encryption and extortion.

-   **Tooling:** DragonForce ransomware, Backdoor.Turn, Mimikatz (credential theft), Impacket (remote execution and lateral movement).
-   **Key ATT&CK techniques:** T1204 User Execution; T1027 Obfuscated Files or Information; T1140 Deobfuscate/Decode Files; T1562 Impair Defenses; T1070 Indicator Removal; T1083 File and Directory Discovery; T1090 / T1090.001 Proxy; T1560 Archive Collected Data; T1567 Exfiltration Over Web Service.

| **![](./Task%202_Report_images/image-003.png)** |
| --- |

*Figure 2: DragonForce Diamond Model view in OpenCTI*

**Associated Indicators**

*OpenCTI holds 69 indicators for DragonForce. A representative sample is shown below (defanged); the full set can be exported from the entity's Indicators view.*

| **Type** | **Indicator** | **Context** |
| --- | --- | --- |
| Domain | relay\[.\]eurofin\[.\]digital | CitrixBleed 2 reporting |
| Domain | relay\[.\]dltsolutions\[.\]top | CitrixBleed 2 reporting |
| Domain | opa\[.\]tlsd\[.\]shop | CitrixBleed 2 reporting |
| Domain | vpts\[.\]us | CitrixBleed 2 reporting |
| Domain | glanz-gmbh\[.\]de | Backdoor.Turn / BYOVD reporting |
| Domain | projetosmecanicos\[.\]com\[.\]br | Backdoor.Turn / BYOVD reporting |
| Tor address | dragonforxxbp3awc7mzs5dkswrua3znqyx5roefmi4smjrsdi22xwqd\[.\]onion | Saudi Arabia report |
| MD5 | e7264a4c331eac851fa75438919e0531 | Saudi Arabia report |
| SHA-1 | e164bbaf848fa5d46fa42f62402a1c55330ef562 | Saudi Arabia report |
| SHA-1 | 0a0a9f2a6772942557ab5355d76af442f8f65e01 | Saudi Arabia report |

Also tied to the group: CVE-2025-5777 (CitrixBleed 2), one of 12 vulnerabilities linked to DragonForce in OpenCTI.

| **![](./Task%202_Report_images/image-004.png)** |
| --- |

*Figure 3: DragonForce indicators recorded in OpenCTI*

**Relevant Campaigns / Groups**

No Campaign objects are linked to DragonForce in OpenCTI; campaign context comes from three AlienVault reports:

-   “DragonForce Ransomware Group is Targeting Saudi Arabia” (Feb 27, 2025) — the direct country-specific report.
-   “Attackers Weaponize Microsoft Teams Relays to Stay Hidden” (Jun 16, 2026).
-   “CitrixBleed 2 (CVE-2025-5777): 7 Steps to DragonForce Ransomware” (Jul 9, 2026).

| **![](./Task%202_Report_images/image-005.png)** |
| --- |

*Figure 4: Reports linked to DragonForce in OpenCTI*

## 2\. Tortoiseshell

**Classification:** Intrusion Set: Iranian-linked cyber-espionage group, active since 2018

**Recorded targeting:** Saudi Arabia, UAE, UK, USA, Canada, Belgium, Australia, Japan; Technology, Aviation and Defense sectors

**Behavior**

Tortoiseshell conducts cyber-espionage focused on defense, aerospace, IT service providers and military organizations. Its most recent reporting shows an enhanced toolset: a reverse SSH tunneling utility disguised as wtsapi32.dll that uses the Windows OpenSSH client to reach its command-and-control server, and a new C++ backdoor (similar to TWOSTROKE) supporting file execution, shell commands, in-memory DLL execution and file manipulation. Domain analysis found subdomains referencing Saudi Arabia and several other countries, indicating a widening of its targeting scope. The group has kept persistent infrastructure despite domain suspensions.

-   **Tooling:** TWOSTROKE-like C++ backdoor; reverse SSH tunnel utility using the Windows OpenSSH client.

| **ATT&CK Tactic** | **Techniques (15 in OpenCTI)** |
| --- | --- |
| Execution | T1059.003 Windows Command Shell; T1574.001 Hijack Execution Flow: DLL |
| Defense Evasion | T1070.004 File Deletion; T1027 Obfuscated Files or Information; T1036.005 Match Legitimate Name or Location |
| Discovery | T1083 File and Directory Discovery; T1082 System Information Discovery |
| Collection | T1005 Data from Local System |
| Command and control | T1090.001 Internal Proxy; T1105 Ingress Tool Transfer; T1071.001 Web Protocols; T1573.002 Asymmetric Cryptography; T1572 Protocol Tunneling; T1132.001 Standard Encoding |
| Exfiltration | T1041 Exfiltration Over C2 Channel |

| **![](./Task%202_Report_images/image-006.png)** |
| --- |

*Figure 5: Tortoiseshell Diamond Model view in OpenCTI*

**Associated Indicators**

| **Type** | **Indicator** | **Context** |
| --- | --- | --- |
| Domain | tiktok-u\[.\]sbs | Backdoor / SSH tunnel report (Aug 2026) |
| Domain | locat\[.\]sbs | Backdoor / SSH tunnel report (Aug 2026) |
| File name | wtsapi32.dll | Legitimate Windows file name used to disguise the SSH tunnel tool |
| Malware family | TWOSTROKE | Backdoor with similarities to the newly reported C++ backdoor |

| **![](./Task%202_Report_images/image-007.png)** |
| --- |

*Figure 6: Tortoiseshell indicators recorded in OpenCTI*

**Relevant Campaigns / Groups**

No Campaign objects are linked to Tortoiseshell in OpenCTI; campaign context comes from one AlienVault report:

-   “Expands Toolset with New Backdoor, SSH Tunnel” (Aug 26, 2026) — the most recent reporting of any threat identified against Saudi Arabia.
-   Malware lineage: TWOSTROKE, the previously documented backdoor the new C++ implant resembles.

| **![](./Task%202_Report_images/image-008.png)** |
| --- |

*Figure 7: AlienVault report describing Tortoiseshell infrastructure referencing Saudi Arabia*

# Sources

All findings are drawn from AlienVault OTX reporting ingested into OpenCTI: the Saudi Arabia country entity, the DragonForce and Tortoiseshell Intrusion Set pages (Knowledge, Indicators, Malware and Diamond Model views), and their linked reports.