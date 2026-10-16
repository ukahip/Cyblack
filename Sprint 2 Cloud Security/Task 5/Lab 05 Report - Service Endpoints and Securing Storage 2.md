# Lab 05: Service Endpoints and Securing Storage

Cloud Security Phase | Cyblack Internship

Lab environment: Azure, East US, resource group AZ500LAB12

## Introduction

This report documents our work on Lab 05 of the Cloud Security phase. The lab demonstrates how to protect an Azure file share so that only one specific subnet can reach it. It uses a service endpoint on the subnet, a storage account firewall, and network security groups (NSGs).

The scenario is a proof of concept with three goals:

-   Keep traffic to Azure Storage on the Azure backbone network by using a service endpoint.
-   Allow only a chosen subnet (Private) to access the storage account.
-   Confirm that a VM outside that subnet (Public) is refused access.

This report covers the architecture, the steps I followed, the problems I met, and what I learned.

## Overview

The lab uses one virtual network with two subnets, each with its own NSG and one Windows Server 2022 VM. A storage account holds a file share named my-file-share.

| Component | Name | Detail |
| --- | --- | --- |
| Resource group | AZ500LAB12 | East US |
| Virtual network | myVirtualNetwork | 10.0.0.0/16 |
| Subnet | Public | 10.0.0.0/24, no service endpoint |
| Subnet | Private | 10.0.1.0/24, service endpoint for Azure Storage |
| NSG | myNsgPrivate | Allow outbound to Storage, deny outbound to Internet, allow inbound RDP |
| NSG | myNsgPublic | Allow inbound RDP |
| Storage account | az500lab12storage01 | Standard, LRS, firewall set to selected networks |
| File share | my-file-share | Backup disabled |
| VM | myVmPrivate | In the Private subnet |
| VM | myVmPublic | In the Public subnet |

Expected outcome:

-   myVmPrivate can map the file share as drive Z: but cannot reach the internet.
-   myVmPublic is denied access to the file share but can reach the internet.

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-001.png)

Lab 05 architecture · 2 subnets, 1 storage account

Figure 1: Both VMs sit in one virtual network, but only the Private subnet has a service endpoint and a path to the storage account.

## Step-by-Step Guide

All resources are created in resource group AZ500LAB12 in the East US region.

### Step 1: Create the virtual network

1.  In the Azure portal, search for Virtual networks and select + Create.
2.  Resource group: create new, AZ500LAB12. Name: myVirtualNetwork. Region: East US.
3.  On the IP addresses tab, set the address space to 10.0.0.0/16.
4.  Edit the default subnet: name it Public, range 10.0.0.0/24, then save.
5.  Select Review + create, then create.

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-002.jpeg)

Screenshot: Virtual network Basics tab and IP addresses tab

### Step 2: Add the Private subnet

1.  Open myVirtualNetwork, go to Settings, then Subnets, and select + Subnet.
2.  Name: Private. Range: 10.0.1.0/24. Leave service endpoints as None for now, then Save.

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-003.jpeg)

Screenshot: Subnets blade showing Public and Private

### Step 3: Create and configure myNsgPrivate

1.  Search for Network security groups and select + Create. Name: myNsgPrivate, resource group AZ500LAB12, region East US. After it has created, click on Goto Resource then Settings
2.  Under Outbound security rules, add the following rules.

| Setting | Rule 1: Allow storage | Rule 2: Deny internet |
| --- | --- | --- |
| Source | Service Tag | Service Tag |
| Source service tag | VirtualNetwork | VirtualNetwork |
| Source port ranges | * | * |
| Destination | Service Tag | Service Tag |
| Destination service tag | Storage | Internet |
| Service | Custom | Custom |
| Destination port ranges | * | * |
| Protocol | Any | Any |
| Action | Allow | Deny |
| Priority | 1000 | 1100 |
| Name | Allow-Storage-All | Deny-Internet-All |

1.  Under Inbound security rules,

| Setting | myNsgPrivate |
| --- | --- |
| Source | Any |
| Source port ranges | * |
| Destination | Service Tag |
| Destination service tag | VirtualNetwork |
| Service | Custom |
| Destination port ranges | 3389 |
| Protocol | TCP |
| Action | Allow |
| Priority | 1200 |
| Name | Allow-RDP-All |

1.  Under Subnets, select + Associate and choose myVirtualNetwork and the Private subnet.

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-004.jpeg)

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-005.jpeg)

Screenshot : myNsgPrivate outbound rules

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-006.jpeg)

Screenshot: myNsgPrivate inbound rules and subnet association

### Step 4: Create and configure myNsgPublic

1.  Create an NSG named myNsgPublic in AZ500LAB12, East US. Click on Goto Resource after it has been created
2.  Click on Inbound Security Rules, Add the inbound rule Allow-RDP-All with the same settings below:

| Setting | myNsgPublic |
| --- | --- |
| Source | Any |
| Source port ranges | * |
| Destination | Service Tag |
| Destination service tag | VirtualNetwork |
| Service | Custom |
| Destination port ranges | 3389 |
| Protocol | TCP |
| Action | Allow |
| Priority | 1200 |
| Name | Allow-RDP-All |

1.  Associate the NSG with the Public subnet:- Click on Subnet, Associate, Virtual Network would be myVirtualNetwork (AZ500LAB12) then Subnet would be Public

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-007.png)

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-008.jpeg)

Screenshot: myNsgPublic rules and subnet association

### Step 5: Create the storage account and file share

1.  Search for Storage accounts and select + Create.
2.  Resource group: AZ500LAB12. Name: a globally unique name (lowercase letters and digits). Region: East US. Performance: Standard. Redundancy: LRS, Primary Service: Azure Blob Storage or Azure Data Lake Storage, Click on Next till you get to Review + Create and click on Create.
3.  Open the account, on the Left-Hand Side Tab, Click on Data Storage, then click on Classic File Shares;
4.  Click on +Classic File Share, Fill in the following details Name: my-file-share.
5.  Click on Backup Tab, Untick Enable backup, Click Next then Create.
6.  Open the share (Click on Data Storage> Classic File Share), Click on **my-file-share**
7.  select Connect, choose the Windows tab, click on show Script and copy the PowerShell script. Store it securely, because it contains the storage account key.

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-009.jpeg)

Screenshot: File share created (do not capture the storage account key)

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-010.png)

Screenshot: File share Script to Map Network Drive

$connectTestResult = Test-NetConnection -ComputerName lab12storage012.file.core.windows.net -Port 445

if ($connectTestResult.TcpTestSucceeded) {

\# Save the password so the drive will persist on reboot

cmd.exe /C "cmdkey /add:\`"lab12storage012.file.core.windows.net\`" /user:\`"localhost\\lab12storage012\`" /pass:\`"<STORAGE_ACCOUNT_KEY>\`""

\# Mount the drive

New-PSDrive -Name Z -PSProvider FileSystem -Root "\\\\lab12storage012.file.core.windows.net\\my-file-share" -Persist

} else {

Write-Error -Message "Unable to reach the Azure storage account via port 445. Check to make sure your organization or ISP is not blocking port 445, or use Azure P2S VPN, Azure S2S VPN, or Express Route to tunnel SMB traffic over a different port."

}

### Step 6: Restrict the storage account to the Private subnet

1.  Open the storage account, then Security + networking, then Networking.
2.  Click on Public Access Tab then navigate to Manage which is below public network access
3.  Set Public network access to Enable (if it’s not already but it will be highlighted on enable already), and then for public network access set to Enabled from selected networks.
4.  Select + Add existing virtual network, choose myVirtualNetwork and the Private subnet only, then click on Enable to complete setup then Click on Add .
5.  Select Save at the bottom of the page.
6.  Verify that the Virtual Networks table lists myVirtualNetwork, Private, with Endpoint Status Enabled.

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-011.jpeg)

Screenshot: Storage Networking page showing the Private subnet with Endpoint Status Enabled

### Step 7: Deploy the two virtual machines

For both VMs, use Windows Server 2022 Datacenter: Azure Edition (Gen 2); Resource Group remains AZ500LAB12;

Availability Option still remains “No Infrastructure Redundancy Required”; Security Type: Standard; VM size: I used **D2nlds\_v6.**

username Student, a password of my own, OS disk Standard HDD, Public inbound ports None, and NIC network security group None (Networking Tab).

Click **on the Networking Tab**, then set:

-   **Virtual network:** myVirtualNetwork
-   **Subnet:** open the dropdown and choose **Private (10.0.1.0/24)**
-   **Public IP:** click **Create new**, name it myVmPrivate-ip, and click OK
-   **NIC network security group:** **None**
-   Do the same for myVmPublic, but choose **Public (10.0.0.0/24)** as the subnet and name the public IP myVmPublic-ip.

| Setting | myVmPrivate | myVmPublic |
| --- | --- | --- |
| Subnet | Private (10.0.1.0/24) | Public (10.0.0.0/24) |
| Public IP | myVmPrivate-ip | myVmPublic-ip |

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-012.jpeg)

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-013.jpeg)

Both VMs in Running state

### Step 8: Test from myVmPrivate

1.  Select Connect on the VM, download the RDP file, and sign in as Student.
2.  Open Windows PowerShell ISE as an Administrator, paste the saved script, and run it.
3.  Open File Explorer and confirm that drive Z: is mapped.
4.  Run Test-NetConnection -ComputerName www.bing.com -Port 80. This should fail.

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-014.jpeg)

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-015.jpeg)

Screenshot: Z: drive mapped in File Explorer

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-016.jpeg)

Screenshot: Test-NetConnection result showing TcpTest Failed

### Step 9: Test from myVmPublic

1.  Connect to the VM through RDP and sign in as Student.
2.  Run the same PowerShell script. It should return New-PSDrive: Access is denied.
3.  Run Test-NetConnection -ComputerName www.bing.com -Port 80. This should succeed.

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-017.jpeg)

Screenshot: Access is denied error on New-PSDrive

![](./Lab%2005%20Report%20-%20Service%20Endpoints%20and%20Securing%20Storage%202_images/image-018.jpeg)

Screenshot: Test-NetConnection result showing TcpTestSucceeded True

### Step 10: Clean up

Open Cloud Shell in PowerShell and run:

Remove-AzResourceGroup -Name "AZ500LAB12" -Force -AsJob

## Glossary

-   **Azure Storage:** Microsoft's cloud storage service, including blobs, files, queues and tables.
-   **Azure file share:** A managed file share in a storage account, accessed over SMB (port 445).
-   **Virtual network (VNet):** A private network in Azure where resources communicate securely.
-   **Subnet:** A range of IP addresses inside a virtual network, used to group and separate resources.
-   **Service endpoint:** A subnet setting that routes traffic to an Azure service over the Azure backbone network and lets that service restrict access to the subnet.
-   **Microsoft.Storage:** The service endpoint type for Azure Storage.
-   **Microsoft.Storage.Global:** A service endpoint type for Azure Storage that applies across regions. A subnet can use only one storage endpoint type at a time.
-   **Network security group (NSG):** A set of inbound and outbound rules that allow or deny traffic to a subnet or network interface.
-   **Service tag:** A named group of IP address ranges managed by Azure, such as Storage, Internet and VirtualNetwork.
-   **Priority (NSG rule):** A number from 100 to 4096 that sets the order of evaluation. The lowest number is processed first.
-   **Storage firewall:** The Networking setting on a storage account that limits access to selected networks or IP addresses.
-   **RDP (Remote Desktop Protocol):** A protocol for connecting remotely to a Windows machine, using TCP port 3389.
-   **SMB (Server Message Block):** The file-sharing protocol used by Azure file shares, using TCP port 445.
-   **Private Endpoint:** A network interface with a private IP address that connects privately to an Azure service.
-   **Defense in depth:** A security approach that layers several controls so that one failure does not expose the system.
-   **Resource group:** A container that holds related Azure resources and can be deleted as a unit.

## Conclusion

This lab showed how a service endpoint and a storage firewall can limit an Azure file share to a single subnet. NSG rules added a second layer by controlling what each subnet can reach. Together they illustrate defense in depth.

The experience taught me to read portal errors carefully and to check the current state of a resource rather than rely on lab instructions that may be out of date.

Key takeaways:

-   Service endpoints keep storage traffic on the Azure backbone and allow subnet-level restrictions.
-   NSG rules are evaluated by priority number, lowest first.
-   Service endpoints are a good start, but Private Endpoints give stronger isolation.
-   Credentials such as storage keys must be protected, and lab resources should be cleaned up.