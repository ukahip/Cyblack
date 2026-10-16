# Lab 07: Configuring Microsoft Defender for Cloud Enhanced Security Features for Servers

## Introduction

This report covers Lab 07 of the Cloud Security Sprint. The scenario is an Azure Security Engineer at a global e-commerce company that runs critical applications, customer data and transactions on Azure virtual machines and on-premises servers.  
  
The Chief Information Security Officer asked for stronger protection against cyber threats, vulnerabilities and misconfigurations.  
  
The task was to enable Microsoft Defender for Servers in Microsoft Defender for Cloud and review the enhanced security features of Defender for Servers Plan 2.

## Overview

The lab was done entirely in the Azure portal, on the subscription Azure subscription 1, using the virtual machine myVM created in Lab 06.

Steps completed:

-   Opened Microsoft Defender for Cloud and went to Management, then Environment settings.
-   Selected the subscription and opened Defender plans.
-   Under Cloud Workload Protection (CWPP), set the Servers plan to On and saved the change.
-   Selected Change plan to review the plan details. The Plan selection pane showed Microsoft Defender for Servers Plan 2 at 15 US dollars per server per month. Its features include Microsoft Defender for Endpoint, vulnerability management, agentless vulnerability, secrets and malware scanning, control plane security alerts, file integrity monitoring and Just-in-time VM access for management ports.
-   Checked that the Servers row showed Plan 2 and a resource quantity of 1 server, which is myVM.

Enabling the Servers plan turns on Defender for Servers Plan 2 for the subscription. This plan is also what makes Just-in-time access available in Lab 08.  

![](./Lab%2007%20Report%20-%20Microsoft%20Defender%20for%20Cloud%20Enhanced%20Security%20for%20Servers_images/image-001.jpeg)

Screenshot: Defender plans page with the Servers row showing Plan 2, On and 1 server  

![](./Lab%2007%20Report%20-%20Microsoft%20Defender%20for%20Cloud%20Enhanced%20Security%20for%20Servers_images/image-002.png)

Screenshot: Plan selection pane showing Defender for Servers Plan 2 and its features.  

![](./Lab%2007%20Report%20-%20Microsoft%20Defender%20for%20Cloud%20Enhanced%20Security%20for%20Servers_images/image-003.png)  
Screenshot: Confirmation that the plan settings were saved

## Finding

What worked

-   The Servers’ plan was switched on and the Plan selection pane confirmed Plan 2.
-   The Servers row counted 1 server, so Defender for Cloud recognized the virtual machine.
-   About ten to twenty minutes after the plan was enabled, the Enable just-in-time option became available on myVM, which confirmed the plan had taken effect.  
      
    ![](./Lab%2007%20Report%20-%20Microsoft%20Defender%20for%20Cloud%20Enhanced%20Security%20for%20Servers_images/image-004.jpeg)

Screenshot: myVM Configuration page with Enable just-in-time available  
  

Challenges

-   The Defender for Cloud Inventory page did not show myVM straight after the plan was enabled. A resource type filter was also narrowing the list. Inventory only listed the Log Analytics workspace at first. This is a delay in how Defender for Cloud assesses new plans and resources, and it was not a failure.
-   The Defender plans page also lists other paid plans, such as Storage and Defender CSPM, with their own prices. It is easy to leave a costly plan on by mistake.

Recommendations

-   Check the Status column on every row of the Defender plans page and keep on only the plans the task needs.
-   Allow time for new plans to propagate before testing features that depend on them.
-   Screenshot the Servers plan showing Plan 2, On and 1 server as evidence.
-   Defender for Servers Plan 2 is a paid plan. Complete the sprint labs promptly and turn the plan off when they are finished.

## Glossary

-   Microsoft Defender for Cloud: Microsoft's service for assessing security posture and protecting cloud and hybrid workloads.
-   Defender for Servers Plan 2: The paid plan with the full set of server protections, including endpoint detection, vulnerability management and Just-in-time access.
-   Cloud Workload Protection (CWPP): The group of Defender plans that protect workloads such as servers, storage, databases and containers.
-   Environment settings: The Defender for Cloud page where plans are turned on or off for each subscription.
-   Just-in-time (JIT) VM access: A feature that keeps management ports closed until access is requested.
-   Inventory: The Defender for Cloud list of resources and their protection status.
-   Resource propagation: The delay before a newly enabled setting takes effect on resources.

## Conclusion

Lab 07 enabled Defender for Servers Plan 2 on the subscription and confirmed that Defender for Cloud counted the virtual machine.  
  
The main lessons were to allow time for a new plan to propagate and to watch the cost of other plans on the same page.  
  
The plan is the base for Lab 08, where Just-in-time access reduces the attack surface of the virtual machine.