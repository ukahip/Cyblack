# Lab 09 - Microsoft Sentinel Report

## Introduction

Lab 09 set up Microsoft Sentinel and proved an automated response works from start to finish. Deleting a Just-in-Time (JIT) access policy on a VM raised a Sentinel incident, and a playbook escalated it from Medium to High within one minute.

Microsoft Sentinel is Microsoft's cloud SIEM and SOAR service. It collects logs, runs detection rules over them, groups alerts into incidents, and can run automated responses called playbooks.

The instructor's guide was written for the Azure portal. Since then, Microsoft has moved much of Sentinel into the Microsoft Defender portal, so several steps were redone with the current interface. Each change is recorded where it happened in the Findings section. Microsoft will retire Sentinel in the Azure portal on 31 March 2027.

## Overview

The lab ran in one subscription and one resource group in East US, on 14 October 2026.

| Item | Value |
| --- | --- |
| Tenant | Default Directory |
| Subscription | Azure subscription 1 |
| Resource group | AZ500LAB131415 |
| Region | East US |
| Log Analytics workspace | lgawignite |
| Virtual machine | myVM, with JIT access enabled (from Labs 06 to 08) |
| Playbook (Logic App) | Change-Incident-Severity |
| Custom analytics rule | Playbook Demo |
| Automation rule | Run Change-Severity Playbook |
| Portals used | Microsoft Defender portal (security.microsoft.com) and Azure portal |

The lab had six tasks:

1.  Onboard Microsoft Sentinel and connect the workspace to the Defender portal.
2.  Connect the Azure Activity data connector.
3.  Create an analytics rule from a built-in template.
4.  Build a playbook that raises incident severity.
5.  Create a custom analytics rule with an automation rule that runs the playbook.
6.  Trigger an incident by deleting the JIT policy on myVM, and confirm the playbook ran.

The detection and response chain worked like this:

1.  A user deletes the JIT access policy on myVM.
2.  Azure writes the action to the Azure Activity log.
3.  The Azure Activity connector sends the event to the lgawignite workspace.
4.  The Playbook Demo rule finds the event and creates an incident at medium severity.
5.  The automation rule runs the Change-Incident-Severity playbook.
6.  The playbook signs in with its managed identity and sets the incident to High with the tag AutoEscalated.

**\[Diagram placeholder: detection and response flow, from the JIT policy deletion through the Activity log, workspace, analytics rule, incident, automation rule and playbook, to the escalated incident\]**

## Findings

All six tasks completed and the playbook escalated every test incident. The steps below follow the order they were done in, with the issues met along the way.

### Task 1: Onboard Microsoft Sentinel

1.  Add Sentinel to the workspace. In the Azure portal, open **Microsoft Sentinel**, click **\+ Create**, select **lgawignite** and click **Add**.

Result: Sentinel opened with the banner "Microsoft Sentinel free trial activated", valid from 9 October 2026 to 9 November 2026 with 10 GB per day free.  
  
The start date of 9 October came from an earlier attempt at this lab on the same workspace.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-001.jpeg)

**Screenshot: Microsoft Sentinel Guides page showing the free trial banner and selected workspace lgawignite**

Issue: opening **Content management > Content hub** in the Azure portal showed "This page was moved to Defender portal, please connect your workspace to the Defender portal".

The menu still listed every page, but the pages themselves had moved.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-002.jpeg)

**Screenshot: Content hub in the Azure portal showing the "page was moved" message.  
**

1.  Connect the workspace to the Defender portal. Sign in to security.microsoft.com, open **System > Settings > Microsoft Sentinel > SIEM workspaces**, click **Connect a workspace**, tick **lgawignite**, set it as the primary workspace and click **Connect**.

Result: lgawignite showed as Connected and Primary, in resource group az500lab131415, East US. After a page refresh, a **Microsoft Sentinel** section appeared in the Defender portal menu with Content hub, Data connectors, Analytics and Automation.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-003.jpeg)

Screenshot: SIEM workspaces page showing lgawignite Connected and Primary

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-004.jpeg)

Screenshot: Defender portal menu showing the Microsoft Sentinel section

### Task 2: Connect the Azure Activity data connector

1.  Install the Azure Activity solution. In the Defender portal, open **Microsoft Sentinel > Content management > Content hub**, search **Azure Activity** and click **Install**.

Result: the Azure Activity solution (version 2.0.0) was installed, with 14 analytics rule templates and 2 hunting queries.

1.  Open the connector. Open **Microsoft Sentinel > Configuration > Data connectors**, select **Azure Activity** and click **Open connector page**. The page opened in the Azure portal.

Result: the connector was already **Connected**, with the last log received 8 minutes earlier (14/10/26, 17:44). The page also said "You don't have subscriptions using the legacy method".

The Azure Policy assignment that streams the Activity log had been created during an earlier attempt, so the policy wizard was not run again. Running it would have created a duplicate assignment.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-005.jpeg)

Screenshot: Azure Activity connector page showing Connected and the last log time

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-006.jpeg)

Screenshot: connector instructions showing no subscriptions on the legacy method

1.  Confirm data is arriving. Click **Go to query** on the connector page, close the Queries hub pop-up, switch the editor to **KQL mode** and run:

AzureActivity  
| take 10

Result: 10 rows came back from lgawignite.

They included the JIT policy being written at 16:25 UTC, a JIT access request at 16:43 UTC, the matching network security group changes and myVM starting.

This confirmed the JIT prerequisite from Lab 08 was in place.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-007.jpeg)

Screenshot: Logs query and the 10 AzureActivity results

### Task 3: Create an analytics rule from a template

1.  Open Analytics. In the Defender portal, open **Microsoft Sentinel > Configuration > Analytics**. The first click left the page on SIEM workspaces, and a second click opened Analytics.
2.  Create the rule from a template. Open the **Rule templates** tab, search **Suspicious number of resource creation**, select **Suspicious number of resource creation or deployment activities**, click **Create rule**, keep the defaults on every tab and click **Save**.

Result: the **Active rules** tab showed 1 rule. It had severity Medium, status Enabled, tactic Impact (technique T1496), and Content hub as its source.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-008.jpeg)

Screenshot: Analytics Active rules tab with the template rule and its details pane

### Task 4: Build the playbook

The instructor's guide deploys a ready-made template file, changeincidentseverity.json, which was not in the sprint documents. The playbook was built by hand instead, with the same result.

1.  Open Automation. In the Defender portal, open **Microsoft Sentinel > Configuration > Automation**. The page now has three tabs: Integration profiles, Automation rules and Playbooks.
2.  Start a playbook. Click the blue **\+ Create** at the top right, then **Logic App playbook > Playbook with incident trigger (Legacy)**. The other options were Integration profile, Automation rule and Generated playbook (New). Microsoft now labels the classic Logic App templates "Legacy", but they still work.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-009.jpeg)

Screenshot: + Create menu showing Playbook with incident trigger (Legacy)

1.  Fill in the Deploy: Playbook Template wizard:

| Field | Value |
| --- | --- |
| Playbook name | Change-Incident-Severity |
| Subscription | Azure subscription 1 |
| Resource group | AZ500LAB131415 |
| Location | (US) East US, set automatically |
| Enable diagnostic settings | Not ticked |

On the Connections tab, Microsoft Sentinel showed "New connection will be configured" and could not be expanded. Click **Next**, then **Create**.

Result: the Playbooks tab listed Change-Incident-Severity as Active, created with Logic Apps, input type Sentinel Incident, at 6:29 PM.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-010.jpeg)

Screenshot: Basics tab of the playbook wizard  

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-011.jpeg)

Screenshot: Playbooks tab showing Change-Incident-Severity

1.  Open the playbook in the Azure portal. Click **Change-Incident-Severity** on the Playbooks tab.

Issue: the Azure portal opened in the wrong directory and showed "You don't have access" with error code 401. The details said the access token came from a different tenant from the one the subscription belongs to. A "No subscriptions" pop-up also appeared. The account has access to two directories, and the link opened the wrong one.

Fix: switch the portal to the right directory. Click the gear icon, open **Directories + subscriptions** and click **Switch** next to Default Directory, the directory that holds the subscription.

The playbook then opened with status Enabled and the definition "1 trigger, 0 actions".

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-012.jpeg)

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-013.jpeg)

Screenshot: 401 wrong issuer error page

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-014.jpeg)

Screenshot: playbook Overview showing 1 trigger, 0 actions

1.  Check how the playbook signs in. Open **Development Tools > Logic app code view**, click inside the editor, press **Ctrl + F** and search **ManagedServiceIdentity**. The setting is only in the full workflow code. Searching from the trigger's own Code view tab returned 0 matches.  
    

Result: line 40 of the workflow code read "authentication": { "type": "ManagedServiceIdentity" } on the connection azuresentinel-Change-Incident-Severity.  

The playbook signs in as itself, so no user password or token is stored in it.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-015.jpeg)

Screenshot: code view with line 40 highlighted

1.  Add the Update incident action. Open **Logic app designer**, click **+** under the Microsoft Sentinel incident trigger, choose **Add an action**, search **Microsoft Sentinel** and select **Update incident**. Set these fields, then click **Save**:

| Field | Value |
| --- | --- |
| Incident ARM id | Incident ARM ID (dynamic content from the trigger) |
| Tags to add | AutoEscalated |
| Severity | High |
| Owner and Status | Left empty, so they stay unchanged |

Result: the designer showed two steps, and the Overview definition changed to "1 trigger, 1 action".

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-016.jpeg)

Screenshot: designer with the trigger and the Update incident panel

1.  Let the playbook edit incidents. Open the **lgawignite** workspace, then **Access control (IAM) > + Add > Add role assignment**.
2.  Choose **Microsoft Sentinel Responder**, set **Assign access to** to **Managed identity**, select the Logic app **Change-Incident-Severity**, and click **Review + assign**.

Result: the role assignment showed Change-Incident-Severity as a Managed identity with Microsoft Sentinel Responder, scoped to this resource only.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-017.jpeg)

Screenshot: lgawignite IAM showing the Responder role on the playbook.

### Task 5: Create the custom rule and the automation rule

1.  Remove the leftover automation rule. Open **Automation > Automation rules**, tick **Run Change-Severity Playbook** and click **Delete**. This rule was created on 9 October, was Inactive, and used the newer "When case is created" trigger with an "Update case" action, so it never ran a playbook.

Result: the list showed "No enhanced automation rules". The page now separates enhanced rules (multi-workspace) from standard rules (single workspace).

**\[Screenshot: Automation rules tab before and after the delete\]**

1.  Start a scheduled rule. Open **Analytics**, click **\+ Create > Scheduled query rule**. On the General tab, set the name to **Playbook Demo**, the tactic to **Initial Access** and leave severity at **Medium**.
2.  Set the rule logic. Paste this query and set the schedule:

AzureActivity  
| where ResourceProviderValue =~ "Microsoft.Security"  
| where OperationNameValue =~ "Microsoft.Security/locations/jitNetworkAccessPolicies/delete"

| Setting | Value |
| --- | --- |
| Run query every | 5 minutes |
| Lookup data from the last | 1 hour |

The query in the instructor's guide had web links pasted into it, which would cause a parse error, so the clean version above was used.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-018.jpeg)

Screenshot: Set rule logic tab with the query and schedule

1.  Keep the incident settings. Leave **Create incidents from alerts triggered by this analytics rule** Enabled and **Alert grouping** Disabled.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-019.jpeg)

Screenshot: Incident settings tab

1.  Add the automation rule. On the Automated response tab, click **\+ Add new** and set:

| Field | Value |
| --- | --- |
| Automation rule name | Run Change-Severity Playbook |
| Trigger | When incident is created |
| Condition | Analytic rule name contains the current rule (set automatically) |
| Action | Run playbook |

The task guide uses the trigger "When alert is created". It was changed to "When incident is created" because the playbook uses the incident trigger, and the two must match.

Issue: the playbook list showed Change-Incident-Severity greyed out with "No Microsoft Sentinel permissions".

Fix: click **Manage playbook permissions**, tick **AZ500LAB131415** and click **Apply**.

The Current permissions tab then listed AZ500LAB131415, but the drop-down only allowed the playbook to be selected after a short wait.

The Task guide's resource group name AZ500LAB1314151 has an extra 1 and was not used.  

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-020.png)

Screenshot: playbook drop-down showing No Microsoft Sentinel permissions

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-021.png)

Screenshot: Manage permissions, Current permissions showing AZ500LAB131415

The grant was checked in **AZ500LAB131415 > Access control (IAM)**. The service principal Azure Security Insights held the role Microsoft Sentinel Automation Contributor on the resource group.  
  
![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-022.jpeg)

Screenshot: resource group IAM showing Azure Security Insights with Microsoft Sentinel Automation Contributor

1.  Select the playbook and save. Choose **Change-Incident-Severity**, click **Apply**, then **Next: Review + create** and **save**.

Result: the Automated response tab listed the rule as order 1, trigger Incident created, action "Run playbook Change-Incident-Severity", status Enabled. Analytics then showed 2 active rules, both Medium and Enabled.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-023.jpeg)

Screenshot: Automated response tab with the automation rule  
![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-024.jpeg)

Screenshot: Analytics showing 2 active rules  
  
  
  

The automation needs two separate permissions:

| Identity | Role | Scope | What it allows |
| --- | --- | --- | --- |
| Azure Security Insights (Sentinel) | Microsoft Sentinel Automation Contributor | AZ500LAB131415 | Starting the playbook |
| Change-Incident-Severity (playbook) | Microsoft Sentinel Responder | lgawignite | Changing incidents |

### Task 6: Trigger the incident and confirm the response

1.  Delete the JIT policy. In the Azure portal, open **Just-in-time VM access**, go to the **Configured** tab, click **...** on the myVM row, choose **Remove** and confirm.

Result: the Activity log showed **Delete JIT Network Access Policies** with status Succeeded on Wednesday 14 October 2026 at 19:58:10 West Africa Time, initiated by the lab user account.  
  
A "Create or Update Network Security Group" entry at the same time showed Azure removing the rules JIT had managed.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-025.jpeg)

Screenshot: Activity log entry and its Summary panel.  

1.  Confirm the event reached Sentinel. In the Defender portal, open **Advanced hunting** and run:

AzureActivity  
| where OperationNameValue =~ "Microsoft.Security/locations/jitNetworkAccessPolicies/delete"  
| project TimeGenerated, Caller, ResourceGroup, ActivityStatusValue

Result: 2 rows came back, Start and Success, both at 7:58:10 PM, caller set to the lab user account, resource group AZ500LAB131415.

The event had reached the workspace within 12 minutes of the deletion.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-026.jpeg)

Screenshot: Advanced hunting query and results

1.  Check the incidents. Open **Investigation & response > Incidents & alerts > Incidents**.

Result: the list held several **Playbook Demo** incidents, all with severity High, the tag AutoEscalated and category Initial access.

**\[Screenshot: incident list showing Playbook Demo incidents\]**

1.  Open one incident. Click **Playbook Demo** with ID 13. Clicking the alert first opens the alert page, so use **View incident page** in its banner.

Result: incident 13 showed severity High, status Active and the tag AutoEscalated.  
  
First and last activity were 7:58:10 PM, the incident was created at 8:11:38 PM and last updated at 8:11 PM.  
The alert inside it stayed at Medium, because the playbook changes the incident only.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-027.png)

Screenshot: alert page showing Medium severity

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-028.png)

Screenshot: incident 13 header showing High and AutoEscalated.  
  
  
  
  
  
  
  
  

1.  Read the incident history. Open the **Activities** tab on incident 13.

| Time | Activity | Performed by |
| --- | --- | --- |
| 8:11 PM | Tags changed to "AutoEscalated" | Playbook-Change-Incident-Severity |
| 8:11 PM | Severity changed from Medium to High | Playbook-Change-Incident-Severity |
| 8:11 PM | Alert Playbook Demo was automatically linked | Microsoft Defender XDR |

All three activities were Completed. The Trigger column shows "Manual" for the playbook's changes because they came through the API. The Performed by column shows they were made by the playbook.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-029.png)

Screenshot: incident 13 Activities tab

1.  Check the playbook runs. In the Azure portal, open **Change-Incident-Severity > Overview > Run history**.

Result: 22 successful runs and 0 failed in the last 24 hours. The runs on screen took 1.53 to 5.55 seconds each, and the last one started at 9:00:29 PM.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-030.png)

Screenshot: playbook Overview with Run history.

Issue: one deletion produced 22 playbook runs, one for each new incident.

  
The rule ran every 5 minutes and looked back 1 hour, and alert grouping was off, so each run found the same 19:58 event again. The runs stopped at about 9:00 PM, one hour after the event.  
  
The fix is to turn on **Suppression** in the rule logic (stop running the query for 1 hour after an alert), or to disable the rule once testing is done.

Two further observations were made on the incident page:

-   The incident graph showed "No entities to display", because the rule does not map any entities, such as the Caller column to an Account entity.
-   A banner said that from 21 October, Cases will be the default experience for incident response. This explains the Cases icon in the Defender portal menu.

### Clean up

Remove the policy assignment and the diagnostic setting before deleting the resource group, so the policy does not try to stream logs to a missing workspace. Run these in Cloud Shell (Bash).

1.  List the Activity log policy assignment:  
    

az policy assignment list --query "\[?contains(displayName,'Activity')\].{name:name, displayName:displayName}" -o table  

1.  Save the assignment name from step 1 in a variable:

POLICY\_NAME="<name from step 1>"  
  
  

1.  Delete the policy assignment:

az policy assignment delete --name "$POLICY\_NAME"

1.  List the subscription diagnostic settings:

az monitor diagnostic-settings subscription list -o table

1.  Save the setting that sends to lgawignite in a variable:

DIAG\_NAME="<name from step 4>"

1.  Delete the diagnostic setting:

az monitor diagnostic-settings subscription delete --name "$DIAG\_NAME" --yes

1.  Delete the lab resource group, which removes myVM, lgawignite, Sentinel, the playbook and the role assignments inside it:

az group delete --name AZ500LAB131415 --yes --no-wait

1.  Return Defender for Servers to the Free tier:

az security pricing create -n VirtualMachines --tier Free

1.  Optional: in **Microsoft Entra ID > Properties**, set **Access management for Azure resources** to **No** to remove the elevated access flagged in the IAM banner.

![](./Lab%2009%20-%20Microsoft%20Sentinel%20Report_images/image-031.png)

Screenshot: resource group deletion confirmed

## Glossary

| Term | Meaning |
| --- | --- |
| Alert | A single detection raised by an analytics rule. |
| Alert grouping | A rule setting that joins related alerts into one incident instead of one incident each. |
| Analytics rule | A saved query Sentinel runs on a schedule to find suspicious activity and raise alerts. |
| Automation rule | A rule that acts on new or updated incidents, for example by running a playbook. |
| Azure Activity log | The subscription log of management actions, such as creating, changing or deleting resources. |
| Azure Policy | The Azure service that applies settings across resources, used here to stream the Activity log to the workspace. |
| Data connector | The link that brings a data source's logs into Sentinel. |
| Defender portal | security.microsoft.com, the portal where Sentinel now runs alongside Microsoft Defender XDR. |
| Entity mapping | A rule setting that tags columns in results as entities such as accounts or hosts, so the incident graph can show them. |
| Incident | A case that groups one or more alerts for investigation. |
| Just-in-Time (JIT) VM access | A Defender for Cloud feature that keeps management ports closed until access is requested for a set time. |
| KQL | Kusto Query Language, the query language used in Sentinel and Log Analytics. |
| Log Analytics workspace | The data store Sentinel reads from and writes to. |
| Logic App | The Azure workflow service that playbooks are built on. |
| Lookback | How far back in time each run of a scheduled rule searches. |
| Managed identity | An identity Azure creates for a resource, so it can sign in without stored passwords. |
| Microsoft Sentinel Automation Contributor | The role that lets Sentinel's own service account start playbooks in a resource group. |
| Microsoft Sentinel Responder | The role that lets an identity read and change incidents. |
| Playbook | A Logic App workflow that Sentinel runs to respond to an incident or alert. |
| SIEM | Security information and event management: collecting logs and detecting threats in them. |
| SOAR | Security orchestration, automation and response: running automated responses to detections. |
| Suppression | A rule setting that stops the query running for a set time after it raises an alert. |
| Tenant (directory) | A Microsoft Entra ID organization. An account can belong to more than one. |

## Conclusion

Lab 09 met its goal. A JIT policy deletion at 19:58 was logged, detected and turned into an incident, and the playbook raised that incident from Medium to High and tagged it AutoEscalated in the same minute it was created. The incident's Activities tab names the playbook as the actor for both changes, and its run history shows 22 successful runs and no failures.

The main lessons from the lab were:

-   The instructor's guide predates Sentinel's move to the Defender portal. Content hub only worked after connecting the workspace there, and some pages, such as Data connectors and the playbook itself, still opened in the Azure portal.
-   Moving between the two portals can land in the wrong directory. A 401 "wrong issuer" error means the portal is signed in to another tenant, and switching directory fixes it.
-   A playbook needs two permissions: one for Sentinel to start it, and one for the playbook to change incidents. Using a managed identity with Microsoft Sentinel Responder on one workspace kept access to the minimum needed.
-   A short run interval with a long lookback creates duplicate incidents for one event. Here one deletion produced 22 incidents, which is the kind of alert noise a SOC must avoid.

Improvements for a real deployment:

-   Turn on suppression or alert grouping on the Playbook Demo rule.
-   Map the Caller column to an Account entity so the incident graph shows who made the change.
-   Plan for Cases becoming the default incident view from 21 October 2026, and for the Azure portal retirement of Sentinel on 31 March 2027.