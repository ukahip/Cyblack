# Lab 06: Log Analytics Workspace, Storage Account and Data Collection Rule (DCR)

Cloud Security Sprint 2 | Azure Monitor and logging | Region: East US | Resource group: AZ500LAB131415

## 1\. Lab scenario

As an Azure Security Engineer at a fintech company, you must improve monitoring and security visibility across the Azure VMs that process financial transactions and customer data.  
  
The CISO wants logs and performance metrics centralized so the security team can detect threats and tune performance.  
  
You will deploy a VM, create a Log Analytics workspace and a storage account, and build a Data Collection Rule (DCR) that sends VM telemetry to the workspace through the Azure Monitor Agent (AMA).

## 2\. Objectives

-   Deploy an Azure virtual machine (myVM)
-   Create a Log Analytics workspace (lgawIgnite)
-   Create an Azure storage account
-   Create a Data Collection Rule (DCR1) that collects performance counters into the workspace

## 3\. Prerequisites

-   Active Azure subscription with the Owner or Contributor role
-   Access to the Azure portal (https://portal.azure.com) and Cloud Shell
-   A VM admin password you have recorded, because Labs 07 to 09 reuse the VM
-   Confirm with your instructor that East US is the region for the class

## 4\. Resource summary

| Resource | Name | Type | Notes |
| --- | --- | --- | --- |
| Resource group | AZ500LAB131415 | Resource group | Shared with Labs 07, 08 and 09 |
| Virtual machine | myVM | Windows VM, Standard\_D2\_v4 | Ports 80 and 3389 opened at creation |
| Virtual network / subnet | myVnet / mySubnet | Networking | Created by New-AzVm |
| Public IP | myPublicIpAddress | Standard SKU | Created by New-AzVm |
| NSG | myNetworkSecurityGroup | Network security group | Created by New-AzVm |
| Log Analytics workspace | lgawIgnite | Workspace | Used later by Sentinel |
| Storage account | strgactignite (or your unique variant) | General-purpose v2, LRS | Name must be globally unique |
| Data Collection Rule | DCR1 | Windows, subscription scope | Sends performance counters to lgawIgnite |

## Exercise 1: Deploy an Azure virtual machine (about 10 minutes)

1.  Sign in to the Azure portal.
2.  Open Cloud Shell (first icon at the top right). Select **PowerShell** if prompted and confirm PowerShell is selected in the upper-left drop-down.
3.  On the Getting started window, keep the default (no storage account required), select your lab subscription, leave **Use an existing private virtual network** unchecked, and click **Apply**.
4.  Create the resource group:

New-AzResourceGroup -Name AZ500LAB131415 -Location 'EastUS'

1.  Register encryption at host:

Register-AzProviderFeature -FeatureName "EncryptionAtHost" -ProviderNamespace Microsoft.Compute  

1.  Create the VM:

New-AzVm -ResourceGroupName "AZ500LAB131415" -Name "myVM" -Location 'EastUS' -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress" -PublicIpSku Standard -OpenPorts 80,3389 -Size Standard\_D2\_v4  

1.  When prompted for credentials, enter user **localadmin** and a strong password of your choice. Record it. Wait for the deployment to finish.
2.  Verify the VM:

Get-AzVM -Name 'myVM' -ResourceGroupName 'AZ500LAB131415' | Format-Table  

Expected result: myVM is listed and ProvisioningState is **Succeeded**.

1.  Close the Cloud Shell pane.  
      
    ![](./Lab%2006%20Report%20-%20Log%20Analytics%20Workspace%2C%20Storage%20Account%20and%20Data%20Collection%20Rule_images/image-001.jpeg)

Screenshot: Cloud Shell output of Get-AzVM showing myVM with ProvisioningState Succeeded

## Exercise 2: Create a Log Analytics workspace (about 10 minutes)

1.  In the portal search box, type **Log Analytics workspaces** and press Enter.
2.  Click **\+ Create**.
3.  On the Basics tab, enter:

| Setting | Value |
| --- | --- |
| Subscription | Your lab subscription |
| Resource group | AZ500LAB131415 |
| Name | lgawIgnite |
| Region | East US |

1.  Click **Review + create**, then **create**.

Expected result: lgawIgnite appears in the resource group.

![](./Lab%2006%20Report%20-%20Log%20Analytics%20Workspace%2C%20Storage%20Account%20and%20Data%20Collection%20Rule_images/image-002.jpeg)

Screenshot: lgawIgnite workspace overview page

## Exercise 3: Create an Azure storage account (about 10 minutes)

1.  In the portal search box, type **Storage accounts** and press Enter.
2.  Click **\+ Create**.
3.  On the Basics tab, enter:

| Setting | Value |
| --- | --- |
| Subscription | Your lab subscription |
| Resource group | AZ500LAB131415 |
| Storage account name | strgactignite (add digits if the name is taken) |
| Region | (US) East US |
| Primary service | Azure Blob Storage or Azure Data Lake Storage Gen 2 |
| Performance | Standard (general-purpose v2) |
| Redundancy | Locally redundant storage (LRS) |

1.  Click **Review + create**, wait for validation, then click **Create**. Deployment takes about 2 minutes.

Tip: storage account names must be 3 to 24 characters, lowercase letters and digits only, and unique across Azure. Record the final name you used.

![](./Lab%2006%20Report%20-%20Log%20Analytics%20Workspace%2C%20Storage%20Account%20and%20Data%20Collection%20Rule_images/image-003.jpeg)

Screenshot: storage account deployment complete

## Exercise 4: Create a Data Collection Rule (about 15 minutes)

1.  In the portal search box, type **Monitor** and press Enter.
2.  Under Settings, click **Data Collection Rules**, then **\+ Create**.
3.  On the Basics tab, enter:

| Setting | Value |
| --- | --- |
| Rule Name | DCR1 |
| Subscription | Your lab subscription |
| Resource Group | AZ500LAB131415 |
| Region | East US |
| Platform Type | Windows |
| Data Collection Endpoint | Leave blank |

1.  Click **Next: Resources >**, then **\+ Add resources**.
2.  In the scope selector, tick the **Subscription** box and click **Apply**.
3.  Click **Next: Collect and deliver >**.
4.  Click **\+ Add data source** and set the Data source type to **Performance Counters**. Keep the defaults (CPU, Memory, Disk and Network, each sampled every 60 seconds).
5.  Go to the **Destination** step. Click **\+ Add destination**, set the Destination type to **Azure Monitor Logs**, confirm your subscription, and choose **lgawIgnite** as the account or namespace.
6.  Click **Add data source** at the bottom of the page to save it.
7.  Click **Review + create**, then **create**.

Note: portal wizard labels and the order of panes change between releases. If your screen differs from these steps, the required outcome is the same: a Windows DCR with a Performance Counters data source and an Azure Monitor Logs destination pointing at lgawIgnite.

![](./Lab%2006%20Report%20-%20Log%20Analytics%20Workspace%2C%20Storage%20Account%20and%20Data%20Collection%20Rule_images/image-004.jpeg)

Screenshot: DCR1 Review + create page showing the data source and destination.

## 5\. Verification

1.  Open **Monitor > Data Collection Rules** and confirm DCR1 is listed.
2.  Open DCR1 > **Resources** and confirm the subscription scope (or myVM) is associated.
3.  Open myVM > **Extensions + applications** and look for the Azure Monitor Windows Agent. It can take several minutes to install.
4.  After about 10 to 15 minutes, open lgawIgnite > **Logs** and run:

Perf  
| where TimeGenerated > ago(30m)  
| summarize count() by ObjectName, CounterName  

Expected result: performance counter rows are returned for myVM.

## 6\. Findings, challenges and recommendations

-   **Scope mismatch:** the scenario asks for security events and system logs, but the lab only configures performance counters. Recommendation: add a Windows Event Logs data source (Security, System and Application) to DCR1 so the workspace holds security telemetry for Defender and Sentinel.
-   **Subscription-wide scope:** applying the DCR to the whole subscription is convenient for a lab but too broad for production. Recommendation: scope DCRs to resource groups or tagged resources.
-   **Open management ports:** New-AzVm with -OpenPorts 80,3389 exposes RDP to the internet. Lab 08 addresses this with Just-in-Time access. Recommendation: enable JIT or use Azure Bastion from the start.
-   **Documentation drift:** the source lab refers to labs 8, 9 and 10 for the shared resource group and to password steps that do not exist in this document. Recommendation: update the references to Labs 07, 08 and 09.
-   **Naming:** the storage account name is globally unique, so expect to adjust it.

## 7\. Cleanup

**Do not delete these resources yet.** Lab 07 (Defender for Cloud), Lab 08 (JIT) and Lab 09 (Sentinel) use myVM, lgawIgnite and the AZ500LAB131415 resource group. After Lab 09, remove everything with:

Remove-AzResourceGroup -Name "AZ500LAB131415" -Force -AsJob

## 8\. Conclusion

I deployed a VM, a Log Analytics workspace, a storage account and a Data Collection Rule that collects performance counters from VMs through the Azure Monitor Agent into a central workspace.