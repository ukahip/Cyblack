# Lab 03 - Azure Firewall Report

## Introduction

This lab deployed an Azure Firewall in front of a workload subnet and proved that it controls outbound traffic. The workload server could reach only www.bing.com and two DNS servers. Every other destination was blocked by the firewall's default deny.

Egress control matters because most attacks need an outbound connection. Malware calls home to a command server, and stolen data has to leave the network somewhere. A firewall that only allows named destinations cuts both of those paths.

## Overview

The lab built one virtual network with three subnets, two Windows Server 2022 VMs and one Azure Firewall.  
A route table sends all internet-bound traffic from the workload subnet to the firewall.

| Subnet | Range | What lives there |
| --- | --- | --- |
| Jump-SN | 10.0.0.0/24 | Srv-Jump, private IP 10.0.0.4, public IP 4.236.131.163, RDP allowed from the internet |
| AzureFirewallSubnet | 10.0.1.0/26 | Test-FW01, private IP 10.0.1.4 |
| Workload-SN | 10.0.2.0/24 | Srv-Work, private IP 10.0.2.4, no public IP, RDP allowed only from inside the VNet |

The traffic path tested was: laptop, RDP to Srv-Jump, RDP to Srv-Work, route table, firewall, internet.

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-001.png)

Diagram placeholder: Test-FW-VN with the three subnets, Srv-Jump, Srv-Work, Test-FW01, the Firewall-route table on Workload-SN, and arrows for the RDP path and the outbound path through the firewall.

### Changes from the original lab

| Original lab | What was used instead | Reason |
| --- | --- | --- |
| \\Allfiles\\Labs\\08\\template.json (Windows Server 2016, Standard\_D2s\_v3) | AZ500LAB08-template-revised.json (Windows Server 2022 Azure Edition) | The original file was not supplied, and Standard\_D2s\_v3 is not available on the free subscription |
| VM size Standard\_D2s\_v3 | Standard\_D2ads\_v7 (2 vCPUs, 8 GB) with the NVMe disk controller | Size available in East US on this subscription; v7 sizes need NVMe |
| Test with Internet Explorer | PowerShell first, then Microsoft Edge | Server 2022 uses Edge; PowerShell gives a clear pass or fail |

## Findings

All seven tasks were completed. The firewall allowed www.bing.com and DNS, and blocked every other site tested with the message "Action: Deny. Reason: No rule matched. Proceeding with default action."

### Task 1: Deploy the lab environment from the template

The template creates the VNet, three subnets, two NSGs, two NICs, one public IP and the two VMs. It is shown in full below.

{  
"$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",  
"contentVersion": "1.0.0.0",  
"parameters": {  
"location": { "type": "string", "defaultValue": "eastus", "metadata": { "description": "Lab region." } },  
"adminUsername": { "type": "string", "defaultValue": "localadmin" },  
"adminPassword": { "type": "secureString", "metadata": { "description": "Password used for both lab VMs." } },  
"vmSize": { "type": "string", "defaultValue": "Standard\_D2als\_v7", "metadata": { "description": "VM size selected from the sizes available in East US." } },  
"diskControllerType": { "type": "string", "defaultValue": "NVMe", "allowedValues": \[ "NVMe", "SCSI" \], "metadata": { "description": "NVMe for v6/v7 sizes (e.g. Standard\_D2als\_v7). Set SCSI if you switch to an older size such as Standard\_D2s\_v3." } }  
},  
"variables": {  
"vnetName": "Test-FW-VN",  
"jumpVmName": "Srv-Jump",  
"workVmName": "Srv-Work",  
"jumpSubnetName": "Jump-SN",  
"firewallSubnetName": "AzureFirewallSubnet",  
"workloadSubnetName": "Workload-SN",  
"jumpNsgName": "Srv-Jump-nsg",  
"workNsgName": "Srv-Work-nsg",  
"jumpNicName": "Srv-Jump-nic",  
"workNicName": "Srv-Work-nic",  
"jumpPipName": "Srv-Jump-PIP"  
},  
"resources": \[  
{  
"type": "Microsoft.Network/networkSecurityGroups", "apiVersion": "2024-05-01",  
"name": "\[variables('jumpNsgName')\]", "location": "\[parameters('location')\]",  
"properties": { "securityRules": \[ { "name": "Allow-RDP-Internet", "properties": { "priority": 100, "access": "Allow", "direction": "Inbound", "protocol": "Tcp", "sourcePortRange": "\*", "destinationPortRange": "3389", "sourceAddressPrefix": "\*", "destinationAddressPrefix": "\*" } } \] }  
},  
{  
"type": "Microsoft.Network/networkSecurityGroups", "apiVersion": "2024-05-01",  
"name": "\[variables('workNsgName')\]", "location": "\[parameters('location')\]",  
"properties": { "securityRules": \[ { "name": "Allow-RDP-From-VNet", "properties": { "priority": 100, "access": "Allow", "direction": "Inbound", "protocol": "Tcp", "sourcePortRange": "\*", "destinationPortRange": "3389", "sourceAddressPrefix": "VirtualNetwork", "destinationAddressPrefix": "\*" } } \] }  
},  
{  
"type": "Microsoft.Network/virtualNetworks", "apiVersion": "2024-05-01",  
"name": "\[variables('vnetName')\]", "location": "\[parameters('location')\]",  
"dependsOn": \[  
"\[resourceId('Microsoft.Network/networkSecurityGroups', variables('jumpNsgName'))\]",  
"\[resourceId('Microsoft.Network/networkSecurityGroups', variables('workNsgName'))\]"  
\],  
"properties": {  
"addressSpace": { "addressPrefixes": \[ "10.0.0.0/16" \] },  
"subnets": \[  
{ "name": "\[variables('jumpSubnetName')\]", "properties": { "addressPrefix": "10.0.0.0/24", "networkSecurityGroup": { "id": "\[resourceId('Microsoft.Network/networkSecurityGroups', variables('jumpNsgName'))\]" } } },  
{ "name": "\[variables('firewallSubnetName')\]", "properties": { "addressPrefix": "10.0.1.0/26" } },  
{ "name": "\[variables('workloadSubnetName')\]", "properties": { "addressPrefix": "10.0.2.0/24", "networkSecurityGroup": { "id": "\[resourceId('Microsoft.Network/networkSecurityGroups', variables('workNsgName'))\]" } } }  
\]  
}  
},  
{  
"type": "Microsoft.Network/publicIPAddresses", "apiVersion": "2024-05-01",  
"name": "\[variables('jumpPipName')\]", "location": "\[parameters('location')\]",  
"sku": { "name": "Standard" },  
"properties": { "publicIPAllocationMethod": "Static", "publicIPAddressVersion": "IPv4" }  
},  
{  
"type": "Microsoft.Network/networkInterfaces", "apiVersion": "2024-05-01",  
"name": "\[variables('jumpNicName')\]", "location": "\[parameters('location')\]",  
"dependsOn": \[  
"\[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))\]",  
"\[resourceId('Microsoft.Network/publicIPAddresses', variables('jumpPipName'))\]"  
\],  
"properties": { "ipConfigurations": \[ { "name": "ipconfig1", "properties": {  
"privateIPAllocationMethod": "Dynamic",  
"subnet": { "id": "\[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('jumpSubnetName'))\]" },  
"publicIPAddress": { "id": "\[resourceId('Microsoft.Network/publicIPAddresses', variables('jumpPipName'))\]" }  
} } \] }  
},  
{  
"type": "Microsoft.Network/networkInterfaces", "apiVersion": "2024-05-01",  
"name": "\[variables('workNicName')\]", "location": "\[parameters('location')\]",  
"dependsOn": \[ "\[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))\]" \],  
"properties": { "ipConfigurations": \[ { "name": "ipconfig1", "properties": {  
"privateIPAllocationMethod": "Dynamic",  
"subnet": { "id": "\[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('workloadSubnetName'))\]" }  
} } \] }  
},  
{  
"type": "Microsoft.Compute/virtualMachines", "apiVersion": "2024-03-01",  
"name": "\[variables('jumpVmName')\]", "location": "\[parameters('location')\]",  
"dependsOn": \[ "\[resourceId('Microsoft.Network/networkInterfaces', variables('jumpNicName'))\]" \],  
"properties": {  
"hardwareProfile": { "vmSize": "\[parameters('vmSize')\]" },  
"osProfile": { "computerName": "\[variables('jumpVmName')\]", "adminUsername": "\[parameters('adminUsername')\]", "adminPassword": "\[parameters('adminPassword')\]", "windowsConfiguration": { "provisionVMAgent": true, "enableAutomaticUpdates": true } },  
"storageProfile": {  
"imageReference": { "publisher": "MicrosoftWindowsServer", "offer": "WindowsServer", "sku": "2022-datacenter-azure-edition", "version": "latest" },  
"osDisk": { "createOption": "FromImage", "managedDisk": { "storageAccountType": "Standard\_LRS" } },  
"diskControllerType": "\[parameters('diskControllerType')\]"  
},  
"networkProfile": { "networkInterfaces": \[ { "id": "\[resourceId('Microsoft.Network/networkInterfaces', variables('jumpNicName'))\]" } \] },  
"diagnosticsProfile": { "bootDiagnostics": { "enabled": true } }  
}  
},  
{  
"type": "Microsoft.Compute/virtualMachines", "apiVersion": "2024-03-01",  
"name": "\[variables('workVmName')\]", "location": "\[parameters('location')\]",  
"dependsOn": \[ "\[resourceId('Microsoft.Network/networkInterfaces', variables('workNicName'))\]" \],  
"properties": {  
"hardwareProfile": { "vmSize": "\[parameters('vmSize')\]" },  
"osProfile": { "computerName": "\[variables('workVmName')\]", "adminUsername": "\[parameters('adminUsername')\]", "adminPassword": "\[parameters('adminPassword')\]", "windowsConfiguration": { "provisionVMAgent": true, "enableAutomaticUpdates": true } },  
"storageProfile": {  
"imageReference": { "publisher": "MicrosoftWindowsServer", "offer": "WindowsServer", "sku": "2022-datacenter-azure-edition", "version": "latest" },  
"osDisk": { "createOption": "FromImage", "managedDisk": { "storageAccountType": "Standard\_LRS" } },  
"diskControllerType": "\[parameters('diskControllerType')\]"  
},  
"networkProfile": { "networkInterfaces": \[ { "id": "\[resourceId('Microsoft.Network/networkInterfaces', variables('workNicName'))\]" } \] },  
"diagnosticsProfile": { "bootDiagnostics": { "enabled": true } }  
}  
}  
\],  
"outputs": {  
"virtualNetworkName": { "type": "string", "value": "\[variables('vnetName')\]" },  
"jumpVmName": { "type": "string", "value": "\[variables('jumpVmName')\]" },  
"workVmName": { "type": "string", "value": "\[variables('workVmName')\]" },  
"workloadSubnet": { "type": "string", "value": "10.0.2.0/24" },  
"firewallSubnet": { "type": "string", "value": "10.0.1.0/26" },  
"jumpPublicIPAddress": { "type": "string", "value": "\[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('jumpPipName')), '2024-05-01').ipAddress\]" }  
}  
}

**Step 1.1: Check which VM sizes are available** (Cloud Shell, Bash)

az vm list-skus -l eastus --resource-type virtualMachines --query "\[?name=='Standard\_D2als\_v7' || name=='Standard\_D2nls\_v6'\].{Size:name, Restricted:restrictions\[0\].reasonCode}" -o table

Result: both sizes were listed with an empty Restricted column, so neither was blocked.

**Step 1.2: Load the template in the portal**

In the portal, search Deploy a custom template, select Build your own template in the editor, click Load file, choose **AZ500LAB08-template-revised.json**, then click Save.

**Step 1.3: Fill in the parameters and deploy**

| Setting | Value |
| --- | --- |
| Resource group | AZ500LAB08 (Create new) |
| Region | East US |
| Admin Username | localadmin |
| Admin Password | own password |
| Vm Size | Standard\_D2ads\_v7 (2 vCPUs, 8 GB) |
| Disk Controller Type | NVMe |

  
Click Review + create, then create.

**Step 1.4: Confirm both VMs are running**

az vm list -g AZ500LAB08 -d --query "\[\].{Name:name, State:powerState, PrivateIP:privateIps, PublicIP:publicIps}" -o table

Result:

Name State PrivateIP PublicIP  
\-------- ---------- ----------- -------------  
Srv-Jump VM running 10.0.0.4 4.236.131.163  
Srv-Work VM running 10.0.2.4

Srv-Work has no public IP, as intended.

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-002.jpeg)

Screenshot placeholder: deployment complete page and the az vm list output

### Task 2: Deploy the Azure Firewall

**Step 2.1: Create the firewall**

In the portal, search Firewalls and click + Create. Use these settings:

| Setting | Value |
| --- | --- |
| Resource group | AZ500LAB08 |
| Name | Test-FW01 |
| Region | East US |
| Availability zone | None |
| Firewall SKU | Standard |
| Firewall management | Use Firewall rules (classic) to manage this firewall |
| Choose a virtual network | Use existing, Test-FW-VN |
| Public IP address | Add new, TEST-FW-PIP |
| Enable Firewall Management NIC | Unticked |

  
Click Review + create, then create. Deployment took about 5 to 10 minutes.

Issue met: the wizard defaulted Choose a virtual network to Create new.

It tried to build a second VNet called Test-FW-VN and showed errors for the name, address space and subnet. Selecting Use existing and picking Test-FW-VN cleared every error, because the firewall then used the existing AzureFirewallSubnet.

**Step 2.2: Confirm the firewall and note its private IP**

az network firewall show -g AZ500LAB08 -n Test-FW01 --query "{State:provisioningState, PrivateIP:ipConfigurations\[0\].privateIPAddress}" -o table

Result:

State PrivateIP  
\--------- -----------  
Succeeded 10.0.1.4

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-003.jpeg)

Screenshot placeholder: Test-FW01 Overview page showing the private and public IP

### Task 3: Create the default route

**Step 3.1: Create the route table**

In the portal, search Route tables and click + Create. Set Resource group AZ500LAB08, Region East US, Name Firewall-route. Click Review + create, then create.

**Step 3.2: Associate the route table with Workload-SN**

Open Firewall-route, go to Settings, Subnets, + Associate. Choose virtual network Test-FW-VN and subnet Workload-SN, then click OK.

**Step 3.3: Add the default route**

Go to Settings, Routes, + Add:

| Setting | Value |
| --- | --- |
| Route name | FW-DG |
| Destination type | IP Addresses |
| Destination IP addresses/CIDR ranges | 0.0.0.0/0 |
| Next hop type | Virtual appliance |
| Next hop address | 10.0.1.4 |

**Step 3.4: Confirm the route**

az network route-table route list -g AZ500LAB08 --route-table-name Firewall-route -o table

Result: FW-DG sends 0.0.0.0/0 to VirtualAppliance 10.0.1.4, provisioning state Succeeded.

**Step 3.5: Confirm the subnet association**

az network vnet subnet show -g AZ500LAB08 --vnet-name Test-FW-VN -n Workload-SN --query routeTable.id -o tsv

Result: the ID returned ends in /routeTables/Firewall-route.

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-004.jpeg)

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-005.jpeg)

Screenshot placeholder: Firewall-route Routes page and Subnets page

### Task 4: Application rule to allow only www.bing.com

**Step 4.1: Create the application rule collection**

Open Test-FW01, go to Settings, Rules (classic), Application rule collection, + Add application rule collection. Set Name App-Coll01, Priority 200, Action Allow.

**Step 4.2: Add the rule in the Target FQDNs table**

| name | Source type | Source | Protocol:Port | Target FQDNs |
| --- | --- | --- | --- | --- |
| AllowGH | IP address | 10.0.2.0/24 | http:80,https:443 | www.bing.com |

Click Add and wait for the firewall update to finish.

Issue met: the rule was first typed into the FQDN tags table, which is for Microsoft's preset groups such as WindowsUpdate.

The source also had spaces in front of it, and http://www.bing.com/ had been typed into the name box. The fix was to delete the FQDN tags row and enter the rule in the Target FQDNs table exactly as shown above.

Result: App-Coll01 saved. The Task 7 tests confirm it works.

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-006.jpeg)

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-007.jpeg)

Screenshot placeholder: application rule collection list showing App-Coll01

### Task 5: Network rule to allow DNS

**Step 5.1: Create the network rule collection**

In Rules (classic), open Network rule collection, + Add network rule collection. Set Name Net-Coll01, Priority 200, Action Allow.

**Step 5.2: Add the rule in the IP Addresses table**

| Name | Protocol | Source type | Source | Destination type | Destination | Ports |
| --- | --- | --- | --- | --- | --- | --- |
| AllowDNS | UDP | IP address | 10.0.2.0/24 | IP address | 209.244.0.3,209.244.0.4 | 53 |

Click Add and wait for the update to finish.

Result: Net-Coll01 saved. The Task 7 DNS test confirms it works.

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-008.jpeg)

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-009.jpeg)

Screenshot placeholder: network rule collection list showing Net-Coll01.

### Task 6: Point Srv-Work at the DNS servers

**Step 6.1: Open the network interface**

Open Srv-Work, go to Networking, Network settings, then click the network interface link Srv-Work-nic.

**Step 6.2: Set the DNS servers**

On the NIC, go to Settings, DNS servers, add 209.244.0.3 and 209.244.0.4, then click Apply. Srv-Work restarts automatically.

Note: the lab document describes an older layout (Networking, then the NIC link, then Custom and Save). The current portal path is the one above.

**Step 6.3: Confirm the DNS settings**

az network nic show -g AZ500LAB08 -n Srv-Work-nic --query dnsSettings.dnsServers -o tsv

Expected result: 209.244.0.3 and 209.244.0.4.

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-010.jpeg)

Screenshot placeholder: Srv-Work-nic DNS servers’ page

### Task 7: Test the firewall

**Step 7.1: Check that the laptop can reach Srv-Jump on RDP** (laptop PowerShell)

Test-NetConnection 4.236.131.163 -Port 3389  

Result: TcpTestSucceeded True.

Issue met: the first RDP attempts failed with "Remote Desktop can't connect to the remote computer". The connection was being made from the laptop straight to Srv-Work.

A test from the laptop to 10.0.2.4 on port 3389 also failed, with source address 172.20.10.2 on the laptop's Wi-Fi.

This is expected: Srv-Work has no public IP and can only be reached from inside the VNet. The fix was to connect to Srv-Jump first.

**Step 7.2: RDP from the laptop to Srv-Jump** (laptop, Win + R)

mstsc /v:4.236.131.163

Sign in as localadmin.

**Step 7.3: RDP from Srv-Jump to Srv-Work** (inside the Srv-Jump session, Start, Run)

mstsc /v:Srv-Work  

The private IP also works: mstsc /v:10.0.2.4. Sign in as localadmin.

**Step 7.4: Confirm the session is on Srv-Work** (Srv-Work PowerShell)

Whoami  

Result: srv-work\\localadmin.

**Step 7.5: Test DNS through the network rule**

Resolve-DnsName www.bing.com  

Result: the name resolved through www-www.bing.com.trafficmanager.net and www.bing.com.edgekey.net to e86303.dscx.akamaiedge.net, IPv4 address 23.212.250.4, plus several IPv6 addresses.

**Step 7.6: Test the allowed site through the application rule**

(Invoke-WebRequest https://www.bing.com -UseBasicParsing).StatusCode

  
Result: 200.

**Step 7.7: Test a site with no rule**

(Invoke-WebRequest http://www.microsoft.com -UseBasicParsing).Content  

Result: "Action: Deny. Reason: No rule matched. Proceeding with default action."

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-011.jpeg)  
  
![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-012.jpeg)

Screenshot placeholder: Srv-Work PowerShell showing the Resolve-DnsName output

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-013.png)

Screenshot placeholder: Srv-Work PowerShell showing StatusCode 200, the Deny message and whoami

**Step 7.8: Repeat the test in a browser**

Inside Srv-Work, open Microsoft Edge with Settings, System, about open beside it showing Device name Srv-Work. Browse to https://www.bing.com, then to apple.com.

Result: www.bing.com loaded normally. apple.com showed "Action: Deny. Reason: No rule matched. Proceeding with default action." Edge tried plain http first, which is why the readable deny page appeared.

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-014.png)

Screenshot placeholder: Edge on Srv-Work loading www.bing.com, About page showing Srv-Work

![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-015.png)

Screenshot placeholder: Edge on Srv-Work showing the Deny message for apple.com

### Clean up

**Step 8.1: Delete the resource group**

az group delete --name AZ500LAB08 --yes --no-wait  

**Step 8.2: Confirm the resource group is gone**

az group list -o table  

Result: AZ500LAB08 no longer appears. Running the delete again returned ResourceGroupNotFound, which confirms it was already removed and the firewall charges had stopped.  
  
![](./Lab%2003%20-%20Azure%20Firewall%20Report_images/image-016.png)

Screenshot placeholder: az group list output without AZ500LAB08, and the ResourceGroupNotFound message

## Glossary

| Term | Meaning |
| --- | --- |
| Application rule | A firewall rule that allows or denies traffic by domain name (FQDN) and protocol, such as https to www.bing.com |
| ARM template | A JSON file that describes Azure resources so they can be deployed the same way every time |
| Azure Firewall | Microsoft's managed, stateful firewall service that filters traffic for a virtual network |
| AzureFirewallSubnet | The subnet name Azure Firewall requires; it must be at least /26 |
| Default deny | Traffic that matches no rule is blocked |
| Egress | Outbound traffic leaving a network |
| FQDN | Fully qualified domain name, the full name of a host, such as www.bing.com |
| FQDN tag | A Microsoft-managed group of domain names for a service, such as WindowsUpdate |
| Jump host | A server that admins connect to first, and from which they reach servers that are not exposed to the internet |
| Network rule | A firewall rule that allows or denies traffic by IP address, port and protocol |
| NSG | Network security group, a set of allow and deny rules applied to a subnet or network interface |
| NVMe | A disk controller type that newer Azure VM sizes (v6 and v7) require |
| RDP | Remote Desktop Protocol, used to sign in to a Windows server remotely on TCP port 3389 |
| Route table | A set of user-defined routes that overrides Azure's default routing for a subnet |
| Virtual appliance | A route next hop type that sends traffic to a device such as a firewall |

## Conclusion

The firewall worked as designed. Srv-Work reached www.bing.com and its two DNS servers, and every other site was denied with "No rule matched".

The 0.0.0.0/0 route made the firewall the only exit for the workload subnet, while traffic inside the VNet, such as RDP from Srv-Jump, was not affected.

The main lessons came from the problems met along the way. The original template and VM size did not work on the free subscription, so a revised template with Standard\_D2ads\_v7 and NVMe was used. Several portal screens no longer match the lab document, including the firewall's virtual network option, the two tables in the application rule form and the path to the DNS servers setting. The RDP failure showed the jump host pattern in practice: Srv-Work cannot be reached from the internet at all.

Recommendations for a production setup:

-   Restrict Srv-Jump-nsg so RDP is allowed only from known IP addresses, or replace the jump host with Azure Bastion or just-in-time VM access.
-   Use a Firewall Policy instead of classic rules, so rules can be managed centrally.
-   Send firewall logs to a Log Analytics workspace to record every allow and deny decision.
-   Consider the Premium SKU for TLS inspection and intrusion detection.
-   Add FQDN tags such as WindowsUpdate where servers need updates, rather than opening wider access.
-   Plan firewall work in one sitting and delete or deallocate it afterwards, because it is billed by the hour while deployed.