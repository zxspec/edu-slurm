# lab 2

## lab 2.3

```
kubectl create -f basic-pod.yaml
kubectl get pod
kubectl get pod basicpod -o wide
. . .
# NAME       READY   STATUS    RESTARTS   AGE   IP          NODE           NOMINATED NODE   READINESS GATES
# basicpod   1/1     Running   0          17m   10.1.71.3   192.168.1.25   <none>           <none>
#                                                 ^^^^^
#                                             pod ip address

# curl <pod_ip_addr> -- read directly from pod
curl 10.1.71.3

kubectl create -f basic-service.yaml
kubectl get services
. . .               
# NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
# basicservice   ClusterIP   10.152.183.164   <none>        80/TCP    12m
#                                ^^^^^
#                          service ip address
# kubernetes     ClusterIP   10.152.183.1     <none>        443/TCP   14m

# curl <service_ip_addr> -- read from service (cluster wide availble)
curl 10.152.183.164
```

## lab 2.4
```
kubectl create -f basic-pod-plus.yaml

kubectl get pod -o wide
. . .
# NAME       READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
# basicpod   2/2     Running   0          16s   10.1.19.129   micro   <none>           <none>
#           ^^^^^
#     2 containers in pod  
```

## lab 2.5
```
# create deployment without files spec in default ns
kubectl create deployment firstpod --image=nginx

kubectl get deployment,pod -o wide

kubectl get ns

kubectl get pod -n default 
kubectl get pod -n kube-system 
kubectl get pod -n container-registry
kubectl get pod -n default 

kubectl get pod --all-namespace

# export deployment to a file
kubectl get deployments firstpod -o yaml > firstpod-deployment.yaml
```
