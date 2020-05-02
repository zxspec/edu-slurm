## kubectl explain

```
kubectl explain deployment.status.availableReplicas
kubectl explain deployment.spec.strategy
```

## ConfigMap

```
kubectl apply -f configmap.yaml -f deployment-with-configmap.yaml
kubecet get po
kubectl exec -it <pod_id> -- /bin/bash

# inside container
cat /etc/nginx/conf.d/default.conf
exit

# on host machine 
kubectl port-forward <pod_id> 8080:80 &
curl 127.0.0.1:8080
. . .
# Handling connection for 8080
# my-deployment-574478ff7d-7j2g5
```

## Secret
```
kubectl create secret generic test --from-literal=testpwd=qwerty
kubectl get secrets test -o yaml
. . .
# apiVersion: v1
# data:
#   testpwd: cXdlcnR5
# kind: Secret

# deccode base64
echo cXdlcnR5 | base64 -d
. . .
# qwerty

kubectl describe pod <my-deployment-with-secret_id>
. . .
# Environment:
#   test:  foo
#   pwd:   <set to the key 'testpwd' in secret 'test'>  Optional: false

kubectl exec -it <my-deployment-with-secret_id> -- /bin/bash
# inside container
env | grep pwd
. . .
# pwd=qwerty
```

## Service

```
kubectl apply -f web-service.yaml
kubect get svc
. . .
# NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
# kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP   30m
# web          ClusterIP   10.152.183.214   <none>        80/TCP    99s
#                             ^^^^^
#                       service ip address

kubectl get endpoints
. . .
# NAME         ENDPOINTS                   AGE
# kubernetes   192.168.1.22:16443          34m
# web          10.1.71.8:80,10.1.71.9:80   4m57s
#                        ^^^^^
#           ip addresses of pods serving web requests

curl 10.152.183.214
. . .
# OK 200kub


kubectl run -it --rm --image amouat/network-utils net-utils bash
# inside container 
curl 10.1.71.8 
curl 10.1.71.9
curl 10.152.183.214

# OR
#curl <service_name>
curl web
...
# my-deployment-5fb96f8d6d-fvg58
# OK 200

curl web
...
my-deployment-5fb96f8d6d-pq5s8
# OK 200

nslookup web
. . .
# Server:         10.152.183.10
# Address:        10.152.183.10#53
# Name:   web.default.svc.cluster.local
# Address: 10.152.183.214

```
    web.default.svc.cluster.local
    ^^^   ^^^
    svc    ns

## Ingress
In order for the Ingress resource to work, the cluster must have an ingress controller running.
```
# microk8s setup
microk8s enable ingress
microk8s status | grep ingress
. . .
# ingress: enabled

curl 127.0.0.1
. . .
# <html>
# <head><title>503 Service Temporarily Unavailable</title></head>
# <body>
# <center><h1>503 Service Temporarily Unavailable</h1></center>
# <hr><center>openresty/1.15.8.1</center>
# </body>
# </html>

# deploy pods and service
# it will be accessible by ip
kubectl get svc web
. . .
# NAME   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
# web    ClusterIP   10.152.183.170   <none>        80/TCP    3m31s
curl 10.152.183.170
. . .
# my-deployment-5fb96f8d6d-6j72g

# micro ip 127.0.0.1
# but sevice is available by ip
curl 127.0.0.1
. . .
# <html>
# <head><title>404 Not Found</title></head>
# <body>
# <center><h1>404 Not Found</h1></center>
# <hr><center>openresty/1.15.8.1</center>
# </body>
# </html>

# while service is accessible via host name
curl micro
. . .
#my-deployment-5fb96f8d6d-6j72g
```

## PV, PVC, Provisioners
PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.
PersistentVolumeClaim (PVC) is a request for storage by a user.
```
#check that microk8s storage addon is enabled
microk8s status | grep storage
# enable storage addon (if required)
microk8s enable storage

# available storage classes
kubectl get storageclasses.storage.k8s.io
# OR
kubectl get sc
. . .

# NAME                          PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
# microk8s-hostpath (default)   microk8s.io/hostpath   Delete          Immediate           false                  5d1h

kubectl get pv
# show info about PVC used to create PV 
kubectl get pv <pv_name> -o yaml | grep -i -A 6 claimRef

kubectl get pvc my-claim -o wide
```

## Practice-Fileshare

### Dockerfile
FYI `set -ex`
- `-e` exit immediately if a command exits with a non-zero status.
- `-x`  print commands and their arguments as they are executed.

Details: http://linuxcommand.org/lc3_man_pages/seth.html

### Kube folder
```
kubectl apply -f .
curl -i micro
curl -i micro/files/

# upload file
curl -i micro/files/ -T confignmap.yaml
curl -i micro/files/
. . .
# HTTP/1.1 200 OK
# Server: openresty/1.15.8.1
# Date: Sat, 02 May 2020 14:03:41 GMT
# Content-Type: text/html
# Transfer-Encoding: chunked
# Connection: keep-alive
# Vary: Accept-Encoding
# 
# <html>
# <head><title>Index of /files/</title></head>
# <body bgcolor="white">
# <h1>Index of /files/</h1><hr><pre><a href="../">../</a>
# <a href="configmap.yaml">configmap.yaml</a>                                     02-May-2020 14:03                 522
# </pre><hr></body>
# </html>

```