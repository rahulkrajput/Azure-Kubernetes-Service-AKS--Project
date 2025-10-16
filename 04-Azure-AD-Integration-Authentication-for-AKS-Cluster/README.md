---
title: Azure AD Integration with Azure AKS for Authentication
description: Azure Active Directory Integration with Azure Kubernetes Service Cluster Admins 
---

# Azure AD Authentication for AKS Cluster Admins

## Pre-requisites
- We should have Azure AKS Cluster Up and Running.
```
# Configure Command Line Credentials for kubectl

# Template
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster 

az aks get-credentials --resource-group aks-rg --name aks-cluster

# Verify Nodes
kubectl get nodes 
kubectl get nodes -o wide

# Get Cluster Information
kubectl cluster-info
```

## Breif Intro
- We can use Azure AD Users and Groups to Manage AKS Clusters
- We can create Admin Users in Azure AD and Associate to Azure AD Group named `kubeadmin` and those users can access Azure AKS Cluster using kubectl. 
- [Three important things](https://docs.microsoft.com/en-us/azure/aks/managed-aad#limitations) we need to remember before making any changes to our existing AKS Clusters
- **Important Note-1:** AKS-managed Azure AD integration can't be disabled
- **Important Note-2:** non-RBAC enabled clusters aren't supported for AKS-managed Azure AD integration
- **Important Note-3:** Changing the Azure AD tenant associated with AKS-managed Azure AD integration isn't supported

![Image](https://github.com/user-attachments/assets/cacc5283-65ee-44a4-87d8-974d30c2e212)



## Step-01: Create Azure AD Group and User in Azure AD 
### Create Azure AD Group 
- Group Type: security 
- Group Name: kubeadmin
- Group Description: AKS Cluster Admins who has full access to Kubernetes Clusters 
- Click on **Create**

### Create Azure AD User & Associate User to Group
- Create User in Azure Active Directory &  Associate User to **kubeadmin** group
- Go to All Services -> Azure Active Directory -> Users -> New User
- **Identity**
  - Username: user01
  - Name: user01 AKSAdmin
  - First Name: user01
  - Last Name: AKSAdmin
- **Password**
  - Let me create the password: check the radio button
  - Initial Password: @KubeAdmin456
- **Groups & Role**
  - Groups: kubeadmin
  - Roles: User
- Rest all leave to defaults
- Click on **Create**

### Complete First Time user Password Change
- Gather Full username from AD
- URL: https://portal.azure.com
- Username: user01@xyz.onmicrosoft.com 
- Current Password: @KubeAdmin456
- New Password: @KubeAdmin123
- Confirm Password: @KubeAdmin123

### Final Username and Password
- Username: user01@xyz.onmicrosoft.com  
- Password: @KubeAdmin123


## Step-02: Enable AKS Cluster with AKS-managed Azure Active Directory feature
- Go to All Services -> Kubernetes Services -> aks-cluster -> Settings -> Configuration
- **AKS-managed Azure Active Directory:** Select **Enabled** radio button
- **Admin Azure AD groups:** kubeadmin
- Click on **SAVE**


## Step-03: Access an Azure AD enabled AKS cluster using Azure AD User
- **Important Note:** Once we do **devicelogin** credentials are cached for all subsequent kubectl commands.
```
# Configure kubectl
az aks get-credentials --resource-group aks-rg --name aks-cluster --overwrite-existing

# View Cluster Information
kubectl cluster-info
URL: https://microsoft.com/devicelogin
Code: D8VE9YE5F (Sample)(View on terminal)
Username: user01@xyz.onmicrosoft.com 
Password: @KubeAdmin123

# List Nodes
kubectl get nodes

# List Pods
kubectl get pods -n kube-system

# List Everything
kubectl get all --all-namespaces
```

## Step-04: How to bypass or Override AD Authentication and use k8s admin?
- If we have issues with AD Users or Groups and want to override that we can use **--admin** to override and directly connect to AKS Cluster
```
# Template
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster --admin

# Replace RG and Cluster Name
az aks get-credentials --resource-group aks-rg --name aks-cluster --admin

# List Nodes
kubectl get nodes

# List Pods
kubectl get pods -n kube-system
```
## Notes

- **Make sure to replace placeholders (e.g., your-Paid-Subscription, your_cluster_name, your_region, your_resource_group_name...etc) with your actual Configuration.**

- **This is a basic setup for demonstration purposes. In a production environment, you should follow best practices for security and performance.**


## References
- [AKS Managed AAD](https://docs.microsoft.com/en-us/azure/aks/managed-aad)
- [Azure AD RBAC](https://docs.microsoft.com/en-us/azure/aks/azure-ad-rbac)
- [Azure AKS Identity](https://docs.microsoft.com/en-us/azure/aks/concepts-identity)