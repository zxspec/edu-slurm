# Lecture 10

https://github.com/southbridgeio/slurm-school-k8s/tree/master/7.Advanced_abstraction

## Daemonset
### Static pods
Put manifest into a /etc/kubernetes/manifests/ and it will launched by kubelet.
Is not controlled by API server.
```
ls -l /etc/kubernetes/manifests/
. . .
-rw-r--r--. 1 root root 3447 Jun  3 00:29 kube-apiserver.manifest
-rw-r--r--. 1 root root 2488 Jun  3 00:29 kube-controller-manager.manifest
-rw-r--r--. 1 root root 2122 Jun  3 00:27 kube-proxy.manifest
-rw-r--r--. 1 root root 1693 Jun  3 00:29 kube-scheduler.manifest
-rw-r--r--. 1 root root 1693 Jun  3 00:29 <put_your_manifest_here>
```

### Pod Anti Affinity
PodAntiAffinity configured to ensure the scheduler does not co-locate replicas on a single node.

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity

### DaemonSet
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

```
kubectl create -f daemonset.yaml 
daemonset.apps/node-name created
kubectl get pod -o wide
. . .
# NAME              READY   STATUS    RESTARTS   AGE   IP         NODE               NOMINATED NODE   READINESS GATES
# node-name-nzhlq   1/1     Running   0          63s   10.0.3.2   k8s-node-1.local   <none>           <none>
# node-name-zz4wb   1/1     Running   0          63s   10.0.1.2   k8s-node-2.local   <none>           <none>
#           ^^^^^
```

Watch logs
```
kubectl logs node-name-zz4wb
. . . 
# k8s-node-1.local
# node-name-4gxjz
# Tue Jun 9 20:33:18 UTC 2020
```

#### Create pods on all nodes
A _daemonset-all-nodes.yaml_ has toleration to a "NoSchedule" taint. It allows to launch daemonset pods even on master and insgress nodes.
```
kubectl describe nodes | grep 'Taints:\|Name:'
. . .
# Name:               k8s-ingress-1.local
# Taints:             node-role.kubernetes.io/ingress:NoSchedule
# Name:               k8s-master-1.local
# Taints:             node-role.kubernetes.io/master:NoSchedule
# Name:               k8s-master-2.local
# Taints:             node-role.kubernetes.io/master:NoSchedule
# Name:               k8s-master-3.local
# Taints:             node-role.kubernetes.io/master:NoSchedule
# Name:               k8s-node-1.local
# Taints:             <none>
# Name:               k8s-node-2.local
# Taints:             <none>
```


```
kubectl apply -f daemonset-all-nodes.yaml

kubectl get po -o wide
. . .
# NAME              READY   STATUS        RESTARTS   AGE   IP          NODE                  NOMINATED NODE   READINESS GATES
# node-name-4n67b   1/1     Running       0          26s   10.0.2.2    k8s-ingress-1.local   <none>           <none>
# node-name-bjwdg   1/1     Running       0          25m   10.0.3.12   k8s-node-1.local      <none>           <none>
# node-name-fp8kj   1/1     Terminating   0          25m   10.0.1.11   k8s-node-2.local      <none>           <none>
# node-name-vdxzf   1/1     Running       0          26s   10.0.5.4    k8s-master-1.local    <none>           <none>
# node-name-vscfw   1/1     Running       0          26s   10.0.0.4    k8s-master-2.local    <none>           <none>
# node-name-wn2vp   1/1     Running       0          26s   10.0.4.3    k8s-master-3.local    <none>           <none>

```

## StatefulSet

### Storage Classes
Check that local-storage class is available, since it is used in an example below.
```
kubectl get storageclasses
. . .
# NAME            PROVISIONER                    AGE
# local-storage   kubernetes.io/no-provisioner   4s
```
Add local-storage class if required
```
kubectl create -f local-storage-class.yaml 
. . .
# storageclass.storage.k8s.io/local-storage created
```
Local volumes do __not__ currently support dynamic provisioning, however a StorageClass should still be created to delay volume binding until Pod scheduling.
Without dynamic provisioning PVC would take all PV space after binding PVC to it. 
For instance, 10GB PV will be used even if PVC is for 1GB.

### Persistant Volume
If PersistantVolume is not available then you would see following error in pod description:
```
kubectl describe pod base-0
. . . 
# Events:
#  ----     ------            ----       ----                       -------
#  ----     ------            ----       ----                       -------
#  Warning  FailedScheduling  <unknown>  default-scheduler          0/6 nodes are available: 2 node(s) didn't find available persistent volumes to bind, 4 node(s) had taints that the pod didn't tolerate.
```
https://stackoverflow.com/questions/52668938/pod-has-unbound-persistentvolumeclaims

Add one more persistent volume
```
kubectl create -f pv-volume-<N>.yaml
```

Scale statefulset
```
kubectl scale statefulset base
```

### Affinity and AntiAffiniti
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

In `statefulset-with-pod-affinity.yaml` a podAntiAffinity is being provided.
The pod anti-affinity rule says that the pod cannot be scheduled onto a node if that node is in the same zone as a pod with label having key "app" and value "base".
`kubernetes.io/hostname` is a node's pre-populated standard label.
https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-hostname

```
kubectl apply -f statefulset-with-pod-affinity.yaml
kubectl get po
. . .
# NAME     READY   STATUS    RESTARTS   AGE
# base-0   1/1     Running   0          7m45s
# base-1   1/1     Running   0          7m43s
# base-2   0/1     Pending   0          7m40s

kubectl describe po base-2
. . .
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/6 nodes are available: 2 node(s) didn't match pod affinity/anti-affinity, 2 node(s) didn't satisfy existing pods anti-affinity rules, 4 node(s) had taints that the pod didn't tolerate.

kubectl describe node k8s-node-1 | grep -i label -A 6
. . .
# Labels:             beta.kubernetes.io/arch=amd64
#                     beta.kubernetes.io/os=linux
#                     kubernetes.io/arch=amd64
#                     kubernetes.io/hostname=k8s-node-1.local
#                                     ^^^
```