# Lab 08: Enable Just-in-Time Access on VMs

Cloud Security Sprint 2

## Introduction

This report covers Lab 08 of the Cloud Security Sprint. The scenario is an Azure Security Engineer at a financial services company. The security team found that management ports left open all the time on virtual machines increase the risk of brute-force attacks and unauthorized access.

The Chief Information Security Officer asked for Just-in-Time (JIT) VM access to be enabled on a virtual machine that processes financial transactions. The task was to enable JIT on the machine and then request access to it.

## Overview

The lab used the virtual machine myVM from Lab 06 and Microsoft Defender for Servers Plan 2 from Lab 07. JIT is a Defender for Cloud feature and does not work without that plan.

Exercise 1: Enable JIT on the VM

-   Opened Virtual machines, selected myVM and opened Configuration under Settings.
-   Selected Enable just-in-time, which led to the Just-in-time VM access page in Microsoft Defender for Cloud.
-   JIT was then enabled for myVM. The default policy for a Windows machine protects port 3389 (RDP), with a maximum request time of three hours.
-   myVM then appeared on the Configured tab, with 0 requests.

![](./Lab%2008%20Report%20-%20Enable%20Just-in-Time%20Access%20on%20VMs_images/image-001.png)

Screenshot: myVM Configuration page showing Enable just-in-time

![](./Lab%2008%20Report%20-%20Enable%20Just-in-Time%20Access%20on%20VMs_images/image-002.jpeg)

\[Screenshot: Just-in-time VM access page with myVM on the Configured tab\]

Exercise 2: Request access to the JIT-enabled VM

-   With JIT on, the management port is closed until access is requested. A request is made from the Just-in-time VM access page, or from the Connect page of the virtual machine.
-   The request specifies the port, the source IP address and the time range. Defender for Cloud then adds a temporary rule to the network security group for that source only.
-   A Remote Desktop connection to myVM on port 3389 was then made successfully. A PowerShell session inside the machine showed its public IP address, 20.119.53.202, which matched the address on the VM overview page.

![](./Lab%2008%20Report%20-%20Enable%20Just-in-Time%20Access%20on%20VMs_images/image-003.jpeg)

![](./Lab%2008%20Report%20-%20Enable%20Just-in-Time%20Access%20on%20VMs_images/image-004.jpeg)

\[Screenshot: Request access pane with port 3389, source IP and time range\]

![](./Lab%2008%20Report%20-%20Enable%20Just-in-Time%20Access%20on%20VMs_images/image-005.jpeg)

\[Screenshot: Temporary JIT rule for port 3389 in the inbound rules of myNetworkSecurityGroup\]

![](./Lab%2008%20Report%20-%20Enable%20Just-in-Time%20Access%20on%20VMs_images/image-006.jpeg)

\[Screenshot: Remote Desktop session to myVM, with the address and port 3389 in the title bar\]

## Finding

What worked

-   JIT was enabled on myVM and the VM was listed on the Configured tab.
-   After an access request was approved, Remote Desktop over port 3389 worked. This shows that JIT controls the port: it is closed by default and opens only on request.
-   The machine also had outbound internet access.

Challenges

-   The first attempt to enable JIT did not register the VM. The Configured tab showed 0 VMs and the Not Configured tab was empty. Enabling JIT a second time made myVM appear. A likely cause is assessment delay after Defender for Servers was turned on, but this was not confirmed.
-   The Defender for Cloud Inventory page did not list the VM straight away, which is related to the same delay.
-   The manual's default JIT policy allows any source IP address, which is a weakness.

![](./Lab%2008%20Report%20-%20Enable%20Just-in-Time%20Access%20on%20VMs_images/image-007.jpeg)

\[Screenshot: Just-in-time VM access page showing 0 VMs on the Configured tab and an empty Not Configured tab, before JIT was enabled the second time\]

Recommendations

-   Restrict the allowed source to My IP or a trusted address range, and shorten the maximum request time, for example to one hour.
-   Protect only the ports that are needed.
-   Close the Remote Desktop session and sign out when finished, and request access again when needed.
-   Monitor changes to JIT policies. In Lab 09, deleting the JIT policy is the event that triggers a Microsoft Sentinel alert, which shows that JIT changes can be used for detection.
-   Confirm that JIT is on the VM before relying on it, because the first enable did not register in this lab.

## Glossary

-   Just-in-Time (JIT) VM access: A Defender for Cloud feature that blocks inbound management ports by default and opens them for a limited time on request.
-   Remote Desktop Protocol (RDP): The protocol used to connect to a Windows machine remotely, on port 3389.
-   Network security group (NSG): A set of rules that allow or deny traffic to Azure resources. JIT adds and removes a temporary rule here.
-   Management port: A port used to administer a machine, such as 3389 for RDP or 22 for SSH.
-   Brute-force attack: Repeated password guessing against an exposed service.
-   Attack surface: All the places where an attacker could try to enter a system.
-   Microsoft Defender for Cloud: The service that provides JIT access and security recommendations.

## Conclusion

Lab 08 enabled Just-in-Time access on myVM and showed that remote access to the machine can be limited to approved, time-limited requests.

This reduces the exposure of port 3389 to brute-force attacks. The main lessons were to verify that JIT is configured, to tighten the default policy, and to monitor changes to it.

The JIT policy created here is also used in Lab 09 to generate a Microsoft Sentinel alert.