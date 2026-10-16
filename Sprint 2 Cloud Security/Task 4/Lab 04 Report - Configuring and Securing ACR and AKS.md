Lab 04 Report: Configuring and Securing ACR and AKS

Oct 12, 2026 | Orion

# Introduction

This report documents AZ-500 Lab 04, Configuring and Securing ACR and AKS. The lab stores a private container image in Azure Container Registry (ACR), runs it on an Azure Kubernetes Service (AKS) cluster, and exposes the same application two ways, publicly and privately, to show how access and exposure are controlled in a cloud environment. It was completed in Azure Cloud Shell (Bash) using the az CLI and kubectl.

# Overview

The lab connects three components. ACR is a private store for the container image. AKS runs the image as a pod on a managed Kubernetes cluster. A load balancer sits in front of the pod and decides who can reach it: the public load balancer gives a public IP, while the internal load balancer gives a private IP reachable only inside the virtual network. AKS pulls the image from ACR using its own Azure identity, so no registry password is stored.

| ![](./Lab%2004%20Report%20-%20Configuring%20and%20Securing%20ACR%20and%20AKS_images/image-001.png)   Screenshot: system architecture diagram showing ACR, AKS, and the public and internal load balancers |
| --- |

The environment used for this lab was as follows.

| **Item** | **Value** |
| --- | --- |
| Resource group | AZ500LAB09 |
| Region | East US |
| Container registry | az5001087978 (Basic SKU), admin access disabled |
| Image | sample/nginx:v1 |
| AKS cluster | MyKubernetesCluster, Kubernetes 1.35.8, 1 node |
| Node size | Standard\_D2nds\_v6 (2 vCPU) |
| Networking | Azure CNI Overlay, network policy None, Standard load balancer |
| Identity | System-assigned managed identity |
| Tooling | Azure Cloud Shell (Bash), az CLI and kubectl |

# Findings

This section is a step-by-step walkthrough of each task: the commands used, the result, and the screenshot captured on completion. All eight tasks were completed successfully in Azure Cloud Shell (Bash).

## Task 1: Create the resource group and ACR

Created the resource group and a Basic-tier registry. ACR names must be globally unique, so the Bash $RANDOM variable was used to generate a unique name.

**Step 1. Create the resource group**

| $ az group create --name AZ500LAB09 --location eastus |
| --- |

**Step 2. Register the required resource providers**

| for ns in Microsoft.Kubernetes Microsoft.KubernetesConfiguration Microsoft.OperationsManagement \\   Microsoft.OperationalInsights Microsoft.ContainerService Microsoft.ContainerRegistry; do   az provider register --namespace $ns   done |
| --- |

**Step 3. Create the registry (globally unique name via $RANDOM)**

| $ az acr create --resource-group AZ500LAB09 --name az500$RANDOM$RANDOM --sku Basic |
| --- |

**Step 4. Store the registry name in a variable (used by later tasks)**

| ACRNAME=$(az acr list --resource-group AZ500LAB09 --query '\[0\].name' -o tsv)   echo $ACRNAME |
| --- |

Result: registry az5001087978 created, with admin access disabled.

| _![](./Lab%2004%20Report%20-%20Configuring%20and%20Securing%20ACR%20and%20AKS_images/image-002.jpeg)_   _Screenshot: ACR az5001087978 listed in resource group AZ500LAB09._ |
| --- |

## Task 2: Build and push the image to ACR

Created a one-line Dockerfile and built the nginx image into the registry.

**Step 1. Create a one-line Dockerfile**

| $ echo FROM nginx &gt; Dockerfile |
| --- |

**Step 2. Build and push the image to ACR**

| $ az acr build --resource-group AZ500LAB09 --image sample/nginx:v1 --registry $ACRNAME --file Dockerfile . |
| --- |

Result: image sample/nginx:v1 present in the repository.

| _![](./Lab%2004%20Report%20-%20Configuring%20and%20Securing%20ACR%20and%20AKS_images/image-003.jpeg)_   _Screenshot: sample/nginx repository showing the v1 tag._ |
| --- |

## Task 3: Create the AKS cluster

The cluster was first attempted in the Azure portal, but the deployment failed. The portal auto-enabled Prometheus monitoring and the cluster onboarding step returned a Conflict error. The failed cluster was deleted and recreated with the CLI, which does not enable that add-on. The lab default VM size (Standard\_DS2\_v2) was also unavailable in East US, so an allowed 2-vCPU size, Standard\_D2nds\_v6, was used.

**Step 1. Create the cluster**

| az aks create -g AZ500LAB09 -n MyKubernetesCluster --location eastus --tier free \\   \--node-count 1 --node-vm-size Standard\_D2nds\_v6 \\   \--network-plugin azure --network-plugin-mode overlay --generate-ssh-keys |
| --- |

**Step 2. Connect to the cluster**

| $ az aks get-credentials --resource-group AZ500LAB09 --name MyKubernetesCluster --overwrite-existing |
| --- |

**Step 3. Verify the node**

| $ kubectl get nodes |
| --- |

Result: the cluster node reported Ready.

## Task 4: Grant AKS access to ACR and its VNet

Attached the registry (granting the AcrPull role), assigned Contributor on the cluster's VNet, and verified the pull permission.

**Step 1. Attach the registry (grants AcrPull)**

| $ az aks update -n MyKubernetesCluster -g AZ500LAB09 --attach-acr $ACRNAME |
| --- |

**Step 2. Assign Contributor on the cluster's VNet**

| NODE\_RG=$(az aks show -g AZ500LAB09 -n MyKubernetesCluster --query nodeResourceGroup -o tsv)   AKS\_VNET\_NAME=$(az network vnet list -g $NODE\_RG --query '\[0\].name' -o tsv)   AKS\_VNET\_ID=$(az network vnet show -g $NODE\_RG -n $AKS\_VNET\_NAME --query id -o tsv)   AKS\_MANAGED\_ID=$(az aks show -g AZ500LAB09 -n MyKubernetesCluster --query identity.principalId -o tsv)   az role assignment create --assignee-object-id $AKS\_MANAGED\_ID --assignee-principal-type ServicePrincipal --role "Contributor" --scope $AKS\_VNET\_ID |
| --- |

**Step 3. Verify the pull permission**

| $ az aks check-acr -g AZ500LAB09 -n MyKubernetesCluster --acr $ACRNAME.azurecr.io |
| --- |

Result: "Your cluster can pull images from az5001087978.azurecr.io".

## Task 5: Deploy the external service

Created nginxexternal.yaml (a Deployment plus a Service of type LoadBalancer), set the image to the ACR name, and applied it. The file below is saved with any text editor or a heredoc in Cloud Shell.

**Manifest: nginxexternal.yaml**

| apiVersion: apps/v1   kind: Deployment   metadata:   name: nginxexternal   spec:   replicas: 1   selector:   matchLabels:   app: nginxexternal   template:   metadata:   labels:   app: nginxexternal   spec:   containers:   \- name: nginxexternal   image: <ACRUniquename>.azurecr.io/sample/nginx:v1   ports:   \- containerPort: 80   \---   apiVersion: v1   kind: Service   metadata:   name: nginxexternal   spec:   type: LoadBalancer   ports:   \- port: 80   selector:   app: nginxexternal |
| --- |

**Step 1. Set the image name in the manifest**

| $ sed -i "s/&lt;ACRUniquename&gt;/$ACRNAME/" nginxexternal.yaml |
| --- |

**Step 2. Apply the manifest**

| $ kubectl apply -f nginxexternal.yaml |
| --- |

**Step 3. Check the pod**

| $ kubectl get pods |
| --- |

Result: pod Running 1/1.

| _![](./Lab%2004%20Report%20-%20Configuring%20and%20Securing%20ACR%20and%20AKS_images/image-004.jpeg)_   _Screenshot: kubectl get pods showing the nginxexternal pod Running._ |
| --- |

## Task 6: Verify external access

Watched the service until the public load balancer assigned an external IP, then opened it in a browser.

**Step 1. Watch for the public IP**

| $ kubectl get service nginxexternal --watch |
| --- |

Result: public IP 172.214.17.203; the browser showed "Welcome to nginx!".

| _![](./Lab%2004%20Report%20-%20Configuring%20and%20Securing%20ACR%20and%20AKS_images/image-005.jpeg)_   _Screenshot: public nginx page "Welcome to nginx!" at http://172.214.17.203._ |
| --- |

## Task 7: Deploy the internal service

Created nginxinternal.yaml with the internal load balancer annotation (service.beta.kubernetes.io/azure-load-balancer-internal: "true"), set the image, and applied it.

**Manifest: nginxinternal.yaml**

| apiVersion: apps/v1   kind: Deployment   metadata:   name: nginxinternal   spec:   replicas: 1   selector:   matchLabels:   app: nginxinternal   template:   metadata:   labels:   app: nginxinternal   spec:   containers:   \- name: nginxinternal   image: <ACRUniquename>.azurecr.io/sample/nginx:v1   ports:   \- containerPort: 80   \---   apiVersion: v1   kind: Service   metadata:   name: nginxinternal   annotations:   service.beta.kubernetes.io/azure-load-balancer-internal: "true"   spec:   type: LoadBalancer   ports:   \- port: 80   selector:   app: nginxinternal |
| --- |

**Step 1. Set the image name in the manifest**

| $ sed -i "s/&lt;ACRUniquename&gt;/$ACRNAME/" nginxinternal.yaml |
| --- |

**Step 2. Apply the manifest**

| $ kubectl apply -f nginxinternal.yaml |
| --- |

**Step 3. Watch for the private IP**

| $ kubectl get service nginxinternal --watch |
| --- |

Result: a private IP, 10.224.0.5, was assigned instead of a public one.

| _![](./Lab%2004%20Report%20-%20Configuring%20and%20Securing%20ACR%20and%20AKS_images/image-006.jpeg)_   _Screenshot: nginxinternal service showing the private IP 10.224.0.5._ |
| --- |

## Task 8: Verify internal access

Opened a shell inside a running pod and requested the internal IP. Replace POD\_NAME with a pod name from kubectl get pods, typed without angle brackets.

**Step 1. Open a shell inside a running pod**

| $ kubectl exec -it POD_NAME -- /bin/bash |
| --- |

**Step 2. Request the internal service from inside the pod**

| $ curl http://10.224.0.5 |
| --- |

Result: the nginx welcome page was returned from inside the cluster. Opening the same IP in a browser times out, proving the internal service is not reachable from the internet.

| _![](./Lab%2004%20Report%20-%20Configuring%20and%20Securing%20ACR%20and%20AKS_images/image-007.jpeg)_   _Screenshot: nginx page returned by curl from inside a pod (10.224.0.5)._ |
| --- |

| _![](./Lab%2004%20Report%20-%20Configuring%20and%20Securing%20ACR%20and%20AKS_images/image-008.jpeg)_   _Screenshot: browser timeout when opening http://10.224.0.5 (security proof)._ |
| --- |

## Clean up

All lab resources were removed afterwards to avoid further cost. Deleting the resource group also removes the node resource group and every resource inside it.

**Step 1. Delete the resource group**

| $ az group delete --name AZ500LAB09 --yes --no-wait |
| --- |

| _![](./Lab%2004%20Report%20-%20Configuring%20and%20Securing%20ACR%20and%20AKS_images/image-009.jpeg)_   _Screenshot: resource group deletion (Deleting, then ResourceGroupNotFound)._ |
| --- |

## Security analysis

The lab demonstrates four security controls and deliberately leaves gaps a production deployment would close.

| **Control demonstrated** | **How it was shown** |
| --- | --- |
| Private registry | Images come only from the organisation's own ACR, so only approved images run |
| Identity-based access | AKS pulls from ACR using its managed identity with AcrPull, so no password is stored |
| Least privilege | AcrPull grants pull-only access, scoped to one registry |
| Exposure control | The same app is public or private depending on one annotation; the internal service is unreachable from the internet |

Gaps left open by the lab:

-   The public service uses plain HTTP with no TLS or web application firewall (WAF).
-   The Kubernetes API server is public, with authorized IP ranges disabled.
-   The cluster uses local accounts instead of Entra ID integrated RBAC.
-   Network policy is set to None, so any pod can reach any other pod.
-   The VNet role granted is Contributor; Network Contributor scoped to the subnet would be tighter.

# Glossary

| **Term** | **Meaning** |
| --- | --- |
| ACR (Azure Container Registry) | A private store for container images, controlled by Azure identities |
| AKS (Azure Kubernetes Service) | Azure's managed Kubernetes service that runs containers for you |
| Container image | A packaged, ready-to-run copy of an application with everything it needs |
| Pod | The smallest unit Kubernetes runs; a wrapper around one or more containers |
| Node | A virtual machine that runs pods |
| Node pool | A group of nodes with the same configuration; a system pool runs Kubernetes' own components |
| Load balancer | The front door that forwards client requests to a healthy pod |
| Managed identity | An Azure identity used by a service to authenticate without a stored password |
| AcrPull | A role that grants pull-only (download) access to a registry |
| Azure CNI Overlay | A networking mode where pod IPs come from a separate overlay, not the VNet address space |
| Network policy | A rule set controlling which pods may talk to each other |

# Conclusion

The lab produced a working but deliberately permissive system.  
  
The same containerized application was deployed both publicly and privately with a single configuration change, showing attack-surface reduction, identity-based access between Azure services, and least-privilege image access.  
  
Internal exposure protects a service from the internet but not from attackers already inside the cluster, so network policies and tighter role scoping matter in production.