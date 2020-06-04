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
```