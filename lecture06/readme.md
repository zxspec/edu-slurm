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
