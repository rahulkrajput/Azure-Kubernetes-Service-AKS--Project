
# Kubernetes RBAC Role & Role Binding with Azure AD on AKS

<img width="1024" height="554" alt="Image" src="https://github.com/user-attachments/assets/c2b40c76-3fa4-4360-a995-266a6d0b2a5c" />

## Brief Intro
- AKS can be configured to use Azure AD for Authentication.
- we can configure Kubernetes role-based access control (RBAC) to limit access to cluster resources based a user's identity or group membership.


## Step-01: Create a Namespace Dev, QA and Deploy Sample Application
```
Here, we are creating two namespaces to check if a user of the Dev namespace group will access the resource of the QA namespace or not, or, while trying to access it, what kind of message or error appears.

# Configure Command Line Credentials for kubectl
az aks get-credentials --name aks-cluster --resource-group aks-rg --admin

# View Cluster Info
kubectl cluster-info

# Create Namespaces dev and qa
kubectl create namespace dev
kubectl create namespace qa

# List Namespaces
kubectl get namespaces

# Deploy Sample Application
kubectl apply -f kube-manifests/01-Application -n dev
kubectl apply -f kube-manifests/01-Application -n qa

# Access Dev Application
kubectl get svc -n dev
http://<public-ip>/

# Access QA Application
kubectl get svc -n qa
http://<public-ip>/
```

## Step-02: Create AD Group, Role Assignment and User for Dev 
```
# Get Azure AKS Cluster Id
AKS_CLUSTER_ID=$(az aks show --resource-group aks-rg --name aks-cluster --query id -o tsv)
echo $AKS_CLUSTER_ID

# Create Azure AD Group
DEV_AKS_GROUP_ID=$(az ad group create --display-name devaksgroup --mail-nickname devaksgroup --query objectId -o tsv)    
echo $DEV_AKS_GROUP_ID

# Create Role Assignment 
az role assignment create \
  --assignee $DEV_AKS_GROUP_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_CLUSTER_ID

# Create Dev User
DEV_AKS_USER_OBJECT_ID=$(az ad user create \
  --display-name "dev user01" \
  --user-principal-name devuser01@xyz.onmicrosoft.com \
  --password %kubeaks123 \
  --query objectId -o tsv)
echo $DEV_AKS_USER_OBJECT_ID  

# Associate Dev User to Dev AKS Group
az ad group member add --group devaksgroup --member-id $DEV_AKS_USER_OBJECT_ID
```

## Step-03: Test Dev User Authentication to Portal
- URL: https://portal.azure.com
- Username: devuser01@xyz.onmicrosoft.com
- Password: %kubeaks123


## Step-04: Update Kubernetes RBAC Role Binding

```
### Get Object Id for devaksgroup AD Group
```
# Get Object ID for AD Group devaksgroup
az ad group show --group devaksgroup --query objectId -o tsv

# Output
c5804cf8-e9ff-1624-18e6-0d73e6dcdae4
```



### Update Kubernetes RBAC Role Binding for Dev Namespace
- Update Azure AD Group **devaksgroup** Object ID in Role Binding
- **File Name:** rolebinding-dev-namespace.yaml
```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-access-rolebinding
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: dev-user-full-access-role
subjects:
- kind: Group
  namespace: dev
  #name: groupObjectId
  name: "c5804cf8-e9ff-1624-18e6-0d73e6dcdae4"  
```

## Step-05: Create Kubernetes RBAC Role & Role Binding for Dev Namespace
```
# As AKS Cluster Admin (--admin)
az aks get-credentials --resource-group aks-rg --name aks-cluster --admin

# Create Kubernetes Role and Role Binding
kubectl apply -f kube-manifests/02-Roles-and-RoleBindings

# Verify Role and Role Binding
kubectl get role -n dev
kubectl get rolebinding -n dev
```

## Step-06: Access Dev Namespace using devuser01 AD User
```
# Overwrite kubectl credentials
az aks get-credentials --resource-group aks-rg --name aks-cluster --overwrite-existing

# List Pods 
kubectl get pods -n dev
- URL: https://microsoft.com/devicelogin
- Code: Q2NGLUQPE (Sample)(View on terminal)
- Username: devuser01@xyz.onmicrosoft.com
- Password: %kubeaks123

# List Services from Dev Namespace
kubectl get svc -n dev

# List Services from QA Namespace
kubectl get svc -n qa

# Forbidden Message should come when we list QA Namespace resources
Error from server (Forbidden): services is forbidden: User "devuser01@xyz.onmicrosoft.com" cannot list resource "services" in API group "" in the namespace "qa"
```

## Step-07: Delete Resources
```
kubectl delete ns dev
kubectl delete ns qa
```

