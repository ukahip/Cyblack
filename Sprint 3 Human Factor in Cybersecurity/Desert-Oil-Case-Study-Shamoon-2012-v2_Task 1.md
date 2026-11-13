Desert Oil Case Study - Shamoon 2012

# Introduction

This case study lists the top human factor threats in the oil and gas industry which includes Desert Oil,  
It then examines the Shamoon incident: its impact, how we responded, and what we should learn from it.

The incident is Shamoon, the August 2012 attack that wiped about 30,000 computers at Saudi Aramco. It reportedly began when one IT employee opened a phishing email.  
  
The case is old, but it involves the company Desert Oil is modelled on, in the same region, and the same malware hit the sector again in 2016 and 2018.

# 1\. Top human factor threats in oil and gas

Six threats matter most to a company like Desert Oil. Each one starts with an employee or contractor: a click on a link, a reused password, a file sent to the wrong person.  
  
Verizon's 2025 Data Breach Investigations Report found a human element in 60% of breaches across all industries.

**Threat**

**How it reaches Desert Oil**

**Evidence**

Spear-phishing

A targeted fake email to IT or engineering staff with a malicious link or attachment

Started the 2012 Aramco attack. Phishing and other social actions appear in 23% of human-element breaches (Verizon, 2025)

Stolen or reused passwords

An old virtual private network (VPN) or contractor account with a leaked password and no multi-factor authentication (MFA)

Colonial Pipeline, 2021. Credential abuse makes up 32% of human-element breaches (Verizon, 2025)

Contractor and supplier weakness

Attackers breach a supplier that holds Desert Oil data or network access

Aramco data leaked through third-party contractors in 2021. Third-party involvement in breaches doubled to 30% (Verizon, 2025)

Fake recruiter lures

A LinkedIn job offer that delivers a malware-laced file

The Iran-linked group APT34 posed as a Cambridge University lecturer to target energy staff

Careless insiders

Missent files, weak passwords, skipped training

65% of US oil and gas security managers in a 2017 Ponemon survey named this their top threat

Malicious insiders

An employee misuses privileged access

15% in the same 2017 Ponemon survey

# 2\. The incident: Shamoon at Saudi Aramco, 2012

The attackers got in around mid-2012 after an IT technician opened a scam email and clicked a bad link.

The attackers waited. At 11:08 local time on 15 August, during Ramadan, with many staff on leave, they launched Shamoon, malware built to wipe computers. It erased files and spread from machine to machine across the corporate network.

A group calling itself the Cutting Sword of Justice claimed the attack and gave political reasons. US officials blamed Iran but offered no public evidence.

The entry route is still disputed. Several reports say an insider with privileged access released the malware. Saudi Arabia's Interior Ministry said no insiders took part. In both versions a person opened the door, either by clicking or by abusing their access.

# 3\. Impact on the Organisation and its stakeholders

Shamoon destroyed or partly wiped about 30,000 workstations (some accounts say 35,000) and erased data on three-quarters of Aramco's corporate PCs. Aramco spent five months building what amounted to a new network. Oil production kept running on its own separate networks.

The Total damage was never disclosed, To recover, we bought about 50,000 hard drives at premium prices in Southeast Asia, and built a security operations Centre and its team from scratch.

**Stakeholder**

**What happened to them**

Employees

Lost email, internet access and their files. Staff went back to pen, paper and fax while Aramco kept the corporate network offline

Saudi public

Aramco stopped selling fuel to domestic tank trucks for a short time, then gave fuel away free. By 1 September, 17 days after the attack, petrol supplies were running low

Leadership

On 29 August the attackers posted Aramco account passwords, including the CEO's, to show they still had access

The Saudi state

An Aramco vice president said the attackers wanted to stop oil and gas reaching local and international markets

Global buyers

Aramco's bulk buying pushed hard drive prices up. Anyone who bought a drive or computer from September 2012 to January 2013 paid slightly more

# 4\. Response evaluation and lessons learned

**Response**

**Evaluation**

Technicians pulled cables from servers worldwide to cut the network off

Worked. It stopped the spread, but it also stopped most business operations

Production ran on separate networks

The strongest part of the response. Oil kept flowing

Main services back by 25 August, full rebuild in five months

Effective, but only because Aramco could afford 50,000 new drives at short notice

Clean-up of accounts

Failed. Four days after services came back, the attackers posted working passwords, including the CEO's

Public communication

Mixed. Aramco said little until operations were back. Markets stayed calm, but partners and the public got little information

Long-term fixes

A new security operations center and a larger security team. Useful, though Aramco built them after the attack

The damage well contained. It recovered slowly and at great expense, and it did not lock the attackers out of its accounts.

# 5\. Lessons

1.  Train the people attackers target first. Shamoon reportedly started with an IT employee, so staff with admin rights need their own phishing drills on top of the yearly course.
2.  Keep security staffed through holidays. The attackers picked Ramadan on purpose. Desert Oil needs monitoring and an on-call responder through Eid and other leave periods.
3.  Keep business and production networks apart. This one control kept Aramco's oil flowing.
4.  Limit and watch privileged access. Give admins only the rights they need, log what they do, and remove access the day someone leaves.
5.  Reset every password after an incident and require MFA. Aramco's attackers posted the CEO's password four days after the clean-up, and the 2018 Saipem attack ran on stolen passwords.
6.  Hold contractors to the same rules. Attackers hit Saipem in 2018, and Aramco data leaked through contractors in 2021.

# Conclusion

Shamoon cost Aramco about 30,000 computers and five months of rebuilding. Separate networks protected oil production, but nothing on the corporate side stopped the first click or the stolen passwords that followed.

Desert Oil's awareness campaign should focus first on IT and admin staff and on contractors, and it should run hardest in the weeks when most of the company is on leave.

# Sources

-   [Shamoon, Wikipedia](https://en.wikipedia.org/wiki/Shamoon)
-   [Shamoon (2012), NATO CCDCOE Cyber Law Toolkit](https://cyberlaw.ccdcoe.org/wiki/Shamoon_\(2012\))
-   [Shamoon was an external attack on Saudi oil production, Infosecurity Magazine](https://infosecurity-magazine.com/news/shamoon-was-an-external-attack-on-saudi-oil)
-   [Throwback Attack: Hackers hit Saudi Aramco, Control Engineering](https://www.controleng.com/?p=123938)
-   [The inside story of the biggest hack in history, CNN Money via WTKR](https://wtkr.com/2015/08/06/the-inside-story-of-the-biggest-hack-in-history/)
-   [Inside the aftermath of the Saudi Aramco breach, Dark Reading](https://www.darkreading.com/cyberattacks-data-breaches/inside-the-aftermath-of-the-saudi-aramco-breach)
-   [Saipem: update on the cyber attack suffered, Saipem](https://www.saipem.com/en/media/press-releases/2018-12-12/saipem-update-cyber-attack-suffered)
-   [Shamoon malware attacks Saipem's network, SOC Prime](https://socprime.com/en/news/shamoon-malware-attacks-saipems-network/)
-   [Aramco operations safe after contractor's data breach, Security Review](https://securityreviewmag.com/?p=22774)
-   [Verizon 2025 DBIR key takeaways, Abnormal](https://abnormal.ai/fr/blog/verizon-2025-dbir-key-takeaways)
-   [Reflections on the 2025 Verizon DBIR, DomainTools](https://www.domaintools.com/?p=33013)
-   [Colonial Pipeline ransomware, Bad.Technology](https://bad.technology/articles/colonial-pipeline-ransomware-shuts-down-us-east-coast-fuel-supply-for-five-days.html)
-   [Study reveals cybersecurity readiness gaps in US oil and gas industry (Ponemon, 2017), Hydrocarbon Processing](https://www.hydrocarbonprocessing.com/news/2017/02/study-reveals-cybersecurity-readiness-gaps-in-us-oil-and-gas-industry)
-   [Iran-linked APT34 LinkedIn campaign, G5 Cyber Security](https://blog.g5cybersecurity.com/iran-apt34-linkedin-malware-2/?amp=1)