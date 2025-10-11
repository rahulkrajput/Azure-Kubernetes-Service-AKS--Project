# Azure AKS - Horizontal Pod Autoscaling (HPA)

## Brief Intro
- How HPA Works?
- How is HPA configured?


![Image](https://github.com/user-attachments/assets/5d67cab7-ac62-4d0f-8379-dd5acd8e7371)

<img width="773" height="503" alt="Image" src="https://github.com/user-attachments/assets/65d2af3b-a1ed-4156-bcd5-94df2dfd6cd0" />

## Step-01: Review Deploy our Application
```
# Deploy
kubectl apply -f kube-manifests/application

# List Pods, Deploy & Service
kubectl get pod
kubect get svc

# Access Application (Only if our Cluster is Public Subnet)
http://<PublicIP-from-Get-SVC-Output>
```

## Step-02: Create a Horizontal Pod Autoscaler resource for the Deployment
- This command creates an autoscaler that targets 20 percent CPU utilization for the deployment, with a minimum of one pod and a maximum of ten pods. 
- When the average CPU load is below 20 percent, the autoscaler tries to reduce the number of pods in the deployment, to a minimum of one. 
- When the load is greater than 20 percent, the autoscaler tries to increase the number of pods in the deployment, up to a maximum of ten
```
# To achieve the above condition, you can use either the imperative or declarative way. Both produce the same result.

# HPA Imperative Way
kubectl autoscale deployment hpa-deployment --cpu-percent=20 --min=1 --max=10

# Or

# HPA Declarative Way (Optional - If you use above imperative command this is just for reference)
kubectl apply -f kube-manifests/hpa-manifest/hpa-manifest.yml

# Describe HPA
kubectl describe hpa/hpa-deployment 

# List HPA
kubectl get horizontalpodautoscaler.autoscaling/hpa-deployment 
```

## Step-03: Create the load & Verify how HPA is working
```
# Generate Load (new Terminal)
kubectl run apache-bench -i --tty --rm --image=httpd -- ab -n 500000 -c 1000 http://hpa-service-nginx.default.svc.cluster.local/ 

# List all HPA
kubectl get hpa

# List specific HPA
kubectl get hpa hpa-deployment 

# Describe HPA
kubectl describe hpa/hpa-deployment 

# List Pods
kubectl get pods
```

## Step-04: Cooldown / Scaledown
- Default cooldown period is 5 minutes. 
- Once CPU utilization of pods is less than 20%, it will starting terminating pods and will reach to minimum 1 pod as configured.


## Step-05: Delete Resources
```
# Delete HPA
kubectl delete hpa hpa-deployment

# Delete Deployment & Service
kubectl delete -f kube-manifests/application 
```
## Declarative Method

## Step-06: Deploy App and HPA Declarative Manifest
```
# Deploy App
kubectl apply -f kube-manifests/application

# Deploy HPA Manifest
kubectl apply -f kube-manifests/hpa-manifest
```

## Step-07: Create the load & Verify how HPA is working
```
# Generate Load
kubectl run apache-bench -i --tty --rm --image=httpd -- ab -n 500000 -c 1000 http://hpa-service-nginx.default.svc.cluster.local/ 

# List all HPA
kubectl get hpa

# List specific HPA
kubectl get hpa hpa-declarative-deploy

# Describe HPA
kubectl describe hpa/hpa-declarative-deploy

# List Pods
kubectl get pods
```


## Step-08: Delete Resources
```
# Delete HPA & Apps
kubectl delete -R -f kube-manifests/


```


## Referencess
- [Azure AKS - Horizontal Pod Autoscaler](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-scale#autoscale-pods)
