Welcome to the Cyber Threat Intelligence Sprint

In this immersive hands-on challenge, you and your team will operate as cyber threat intelligence analysts within a simulated organization. Your mission is to investigate real-world threats using OpenCTI and the AlienVault OTX connector, identify high-risk actors and activities, and report your findings in a way that supports critical decision-making.

During this scenario, you will complete the following:

- Sector & Threat Landscape Profiling: Determine your organization’s operational sector within OpenCTI and conduct threat information gathering and landscape analysis focused on that sector.

- Country-Level Threat Assessment: Identify the top two cyber threats actively targeting the country where your organization’s headquarters is based, gaining geopolitical insight into threat actor behavior.

- Victim and Threat Deep Dive: Investigate the three most targeted entities over the last three months and analyze the top threats facing them. Apply frameworks such as the Diamond Model, Kill Chain, and timeline analysis to understand attacker behavior and impact.

- Political Threat Actor Analysis: Profile a recent threat actor group with political motivations, uncovering their objectives, tools, techniques, and geographic scope.

- Reporting and Executive Presentation: Develop a detailed intelligence report and deliver a high-level executive briefing that communicates findings clearly and effectively to stakeholders. Your presentation should bridge the gap between technical threat data and business decision-making.

By the end of this sprint, you will have strengthened your skills in threat analysis, intelligence synthesis, and strategic communication. Use OpenCTI and collaborative insights to uncover and explain the evolving threat landscape, ensuring your organization is equipped to proactively defend against adversaries.

Helpful links to familiarize yourself with threat intelligence:

- <https://www.verizon.com/business/resources/T8e2/reports/2024-dbir-data-breach-investigations-report.pdf>

- [2024 Data Breach Investigations Report](https://www.verizon.com/business/resources/reports/dbir/)

- [Ransom-DB - Ransomware Tracking](https://www.ransom-db.com/)

- [CrowdStrike](https://go.crowdstrike.com/2023-global-threat-report?utm_campaign=brand&utm_content=crwd-brand-eur-uki-en-psp-x-trl-x-tct-x_x_x_reports-x&utm_medium=sem&utm_source=bing&utm_term=crowdstrike%20threat%20report&cq_cmp=446050631&cq_plac=&msclkid=0b21e2f3b0df16d5a40b84c4ffec5a6a#hero1_gartner_formholder)

- [<i> What is Cyber Threat Intelligence? \[Beginner's Guide\]](https://www.crowdstrike.com/cybersecurity-101/threat-intelligence/)

- [What is Cyber Threat Intelligence?](https://www.cisecurity.org/insights/blog/what-is-cyber-threat-intelligence)

- [Threat Landscape](https://www.enisa.europa.eu/topics/cyber-threats/threats-and-trends)

- [Threat Modeling \| OWASP Foundation](https://owasp.org/www-community/Threat_Modeling)

Scenario Overview

Your team has been tasked with conducting a focused threat intelligence assessment using OpenCTI as your analysis platform. You will enrich your OpenCTI instance with live community threat data using the AlienVault Open Threat Exchange (OTX) connector and use that intelligence to map out recent threats relevant to your organization’s sector and headquarters location.

# Task 1: Industry Sector Focus & Threat Landscape (Company Being Studied is Saudi Aramco and the Dummy Name for this Internship task for Team 6 is Desert Oil)

> Based on your organization’s purpose, identify and select the most appropriate sector that represents its operation within OpenCTI.
>
> Once identified, carry out threat information gathering and threat landscape analysis on that sector. Your analysis should include:
>
> The top threats, actors, or campaigns affecting the sector.
>
> Commonly observed malware/tools and techniques (e.g., MITRE ATT&CK TTPs).
>
> Patterns of targeting (e.g., preferred victims, initial access methods).
>
> Any notable industry alerts, warnings, or historic incidents in the sector.

# Task 2: National Threat Landscape

- Determine the top 2 threats currently targeting the country where your organization is headquartered.

- Describe each threat briefly, including its behavior, associated indicators, and any relevant campaigns or groups.

> Helpful links (there are more out there. You don’t have to be limited to this list)

- [MITRE ATT&CK®](https://attack.mitre.org/)

- [Cyber Kill Chain®](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)

- [<i>IOCParser - Free IOC Extracting Service](https://iocparser.com/)

- [Ransom-DB - Ransomware Groups](https://www.ransom-db.com/ransomware-groups)

- <https://twitter.com/RansomwareNews>

- Open a free account for Hack Notice (<https://app.hacknotice.com/#/auth/login> )

- Check different hacking group latest victim and correlate them to your company <https://app.hacknotice.com/#/hackers>

- [CVE - CVE](https://cve.mitre.org/)

- <https://www.balbix.com/insights/whats-the-difference-between-cve-and-cvss/>

# Task 3: Victim Profile & Threat Mapping

> Identify the 3 most recently targeted victims (within the last 3 months) in OpenCTI.

- For each victim, describe:

  - The top threat associated with the attack.

  - A brief overview of the threat actor, malware/tool, or campaign involved.

  - The threat’s breakdown using the Diamond Model (Adversary, Capability, Infrastructure, Victim).

  - The timeline of the attack (if available).

  - The stage(s) the threat aligns with on the Global Kill Chain.

>  

- - [Reconnaissance](https://attack.mitre.org/tactics/TA0043/)

  - [Resource Development](https://attack.mitre.org/tactics/TA0042/)

  - [Initial Access](https://attack.mitre.org/tactics/TA0001/)

  - [Execution](https://attack.mitre.org/tactics/TA0002/)

  - [Persistence](https://attack.mitre.org/tactics/TA0003/)

  - [Privilege Escalation](https://attack.mitre.org/tactics/TA0004/)

  - [Defense Evasion](https://attack.mitre.org/tactics/TA0005/)

  - [Credential Access](https://attack.mitre.org/tactics/TA0006/)

  - [Discovery](https://attack.mitre.org/tactics/TA0007/)

  - [Lateral Movement](https://attack.mitre.org/tactics/TA0008/)

  - [Collection](https://attack.mitre.org/tactics/TA0009/)

  - [Command and Control](https://attack.mitre.org/tactics/TA0011/)

  - [Exfiltration](https://attack.mitre.org/tactics/TA0010/)

  - [Impact](https://attack.mitre.org/tactics/TA0040/)

>  
>
> Useful Links

- [MITRE ATT&CK®](https://attack.mitre.org/)

- Sample Mapping - [<img src="./media/image3.wmf" style="width:0.16667in;height:0.16667in" />Leaked Tools TTPs and IOCs Used by Conti Ransomware Group](https://www.picussecurity.com/resource/leaked-tools-ttps-and-iocs-used-by-conti-ransomware-group)

# Task 4: Politically Motivated Threat Group

- In OpenCTI, Identify the most recent politically driven threat actor group with high-profile activity.

- Analyze:

  - Their motivation, targets, and tactics.

  - Any known tools or campaigns they are associated with.

  - Summary of their impact or publicly known incidents.

# Task 5: Report and Executive Presentation

> The board of directors needs to understand the results of your threat intelligence exercise.
>
> Create a slide presentation based on your report to be presented to the board.
>
> Ensure that the content is appropriate for the audience.
>
> 7 minutes for the presentation
