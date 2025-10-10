# Azure AKS - Cluster Autoscaler

## Brief Intro
- The Kubernetes Cluster Autoscaler automatically adjusts the number of nodes in your cluster when pods fail to launch due to lack of resources or when nodes in the cluster are underutilized and their pods can be rescheduled onto other nodes in the cluster.




![Azure AKS - Cluster Autoscaler](https://github.com/user-attachments/assets/240557db-ec90-45ca-92af-ad54a9784d28)

## Step-01: Create Cluster with Cluster Autoscaler Enabled
```
# Setup Environment Variables
export RESOURCE_GROUP=aks-autoscale-rg
export REGION=centralindia
export AKS_CLUSTER=aks-autoscaled-cluster
echo $RESOURCE_GROUP, $REGION, $AKS_CLUSTER

# Create Resource Group
az group create --location ${REGION} \
                --name ${RESOURCE_GROUP}

# Create AKS cluster and enable the cluster autoscaler
az aks create --resource-group ${RESOURCE_GROUP} \
              --name ${AKS_CLUSTER} \
              --enable-managed-identity \
              --generate-ssh-keys \
              --node-count 1 \
              --enable-cluster-autoscaler \
              --min-count 1 \
              --max-count 5 

# Configure Credentials
az aks get-credentials --name ${AKS_CLUSTER}  --resource-group ${RESOURCE_GROUP} 

# List Nodes
kubectl get nodes
kubectl get nodes -o wide

# Cluster Info
kubectl cluster-info

# kubectl Config Current Context
kubectl config current-context
```

## Step-02: We can even enable autoscaling from here on Portal Management console
### Cluster: aks-autoscaled-cluster
- Go to All Services -> Kubernetes Services -> aksdemo2 -> Settings -> Nodepools 
- Select Scale
- Scale Method: Automatic (Nothing but --enable-cluster-autoscaler)
- Node Count Rage: 1 to 5 (Nothing but what we defined in Min and Max)
- Current Node Count: 1 (Nothing but what we defined in Node Count)

## Step-03: Review & Deploy Sample Application
```
# Deploy Application
kubectl apply -f kube-manifests/

# List Pods
kubectl get pods

# Access Application
kubectl get svc
http://<PublicIP-from-get-svc-output>/hello
curl -w "\n" http://52.154.217.196/hello
```

## Step-04: Scale our application to 20 pods
- In 2 to 3 minutes, one after the other new nodes will added and pods will be scheduled on them. 
- Our max number of nodes will be 5 which we provided during cluster creation.
```
# Scale UP the demo application to 20 pods
kubectl get pods
kubectl get nodes 
kubectl scale --replicas=20 deploy ca-app-deployment
kubectl get pods

# Verify nodes
kubectl get nodes -o wide

# Access Application
kubectl get svc
http://<PublicIP-from-get-svc-output>/hello
curl -w "\n" http://<PublicIP-from-get-svc-output>/hello
```
## Step-05: Cluster Scale DOWN: Scale our application to 1 pod
- It might take 5 to 20 minutes to cool down and come down to minimum nodes which will be 2 which we configured during nodegroup creation
```
# Scale down the demo application to 1 pod
kubectl scale --replicas=1 deploy ca-app-deployment

# Verify nodes
kubectl get nodes -o wide
```

## Step-06: Delete Resources 

```
# Delete Apps
kubectl delete -f kube-manifests/

```
## Notes

- **Make sure to replace placeholders (e.g., your-Paid-Subscription, your_cluster_name, your_region, your_resource_group_name...etc) with your actual Configuration.**

- **This is a basic setup for demonstration purposes. In a production environment, you should follow best practices for security and performance.**

## References
- [Azure AKS Cluster Autoscaler](https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler)
