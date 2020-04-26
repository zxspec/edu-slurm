# Pod
```
kubectl create -f my-pod.yaml
kubectl get pod
kubectl describe pod my-pody
```

## MicroK8s uses containerd instead of docker
```
  kubectl describe pod my-pod | grep -i container
  ...
  Containers:
    Container ID:   containerd://d64b54d3c078d0f92601afaf97b39380efa71b2f22b0d84ab071e4c826778a8b
```

## cleanup
```
kubectl delete -f my-pod.yaml 
```

# ReplicaSet
```
kubectl create -f replicaset.yaml 

kubectl get replicasets
# OR
kubectl get rs

# watch
kubectl get rs -w


# scale up to 5
kubectl scale replicaset web --replicas=5

# scale to down 2 
kubectl scale replicaset web --replicas=2
# or
kubectl apply -f replicaset-2-pods.yaml


# total cleanup
kubectl delete all --all

```

# Deployments
```
kubectl create -f deployment.yaml 
kubectl apply -f deployment-2.yaml 
```

## update deployed version
```
kubectl set image deployment web-deployment '*=nginx:1.17.10'
# verify 
kubectl describe deployments web-deployment | grep -i image
```
## rollback
```
# deployed versions list
kubectl rollout history deployment <deployment_name>

# rollback deployment to version 0 (if still present in history ) 
kubectl rollout undo deployment web-deployment --to-revision=0E

```

## cleanup
```
 kubectl delete deployments web-deployment
```

## QoS
### BestEffort
```
kubectl describe pod web-deployment-<XXXXX> | grep QoS
. . .
# QoS Class:       BestEffort => no limits defined
```
### Burstable
```
kubectl describe pod web-deployment-<XXXXX> | grep QoS
. . .
# QoS Class:       Burstable  => request != limit
```
### Guaranteed
```
kubectl describe pod web-deployment-<XXXXX> | grep QoS
. . .
# QoS Class:       Guaranteed => limit == request
```
## Probes
```
kubectl logs <pod_name>
```
# Extra
View node used resources
```
kubectl describe node
. . .
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests         Limits
  --------           --------         ------
  cpu                1 (25%)          900m (22%)
  memory             1485200Ki (12%)  1587600Ki (13%)
  ephemeral-storage  0 (0%)           0 (0%)
  hugepages-1Gi      0 (0%)           0 (0%)
  hugepages-2Mi      0 (0%)           0 (0%)
Events:              <none>
```

Hotkey to clear screen in Git Bash for Windows is `Ctrl + l`