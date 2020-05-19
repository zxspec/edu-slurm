## Service is a set of iptables rules

```
kubectl get svc web 
. . .
# NAME   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
# web    ClusterIP   10.152.183.248   <none>        80/TCP    56s

sudo iptables -t nat -S | grep web
. . .
# -A KUBE-SERVICES ! -s 10.152.183.0/24 -d 10.152.183.248/32 -p tcp -m comment --comment "default/web: cluster IP" -m tcp --dport 80 -j # KUBE-MARK-MASQ
# -A KUBE-SERVICES -d 10.152.183.248/32 -p tcp -m comment --comment "default/web: cluster IP" -m tcp --dport 80 -j # KUBE-SVC-BIJGBSD4RZCCZX5R

sudo iptables -t nat -S | grep KUBE-SVC-BIJGBSD4RZCCZX5R
. . .
# -N KUBE-SVC-BIJGBSD4RZCCZX5R
# -A KUBE-SERVICES -d 10.152.183.248/32 -p tcp -m comment --comment "default/web: cluster IP" -m tcp --dport 80 -j KUBE-SVC-BIJGBSD4RZCCZX5R
# -A KUBE-SVC-BIJGBSD4RZCCZX5R -m comment --comment "default/web:" -j KUBE-SEP-WZRGJPBEBHEXO3BB

sudo iptables -t nat -S | grep KUBE-SEP-WZRGJPBEBHEXO3BB
. . .
# -N KUBE-SEP-WZRGJPBEBHEXO3BB
# -A KUBE-SEP-WZRGJPBEBHEXO3BB -s 10.1.19.193/32 -m comment --comment "default/web:" -j KUBE-MARK-MASQ
#                                     ^^^^^
#                                pod ip-address
# -A KUBE-SEP-WZRGJPBEBHEXO3BB -p tcp -m comment --comment "default/web:" -m tcp -j DNAT --to-destination 10.1.19.193:80
# -A KUBE-SVC-BIJGBSD4RZCCZX5R -m comment --comment "default/web:" -j KUBE-SEP-WZRGJPBEBHEXO3BB

kubectl get po -o wide
. . .
# NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
# my-pod-2                    1/1     Running   4          17d   10.1.19.193   micro   <none>           <none>
```

## Check Kubelet service status
```
systemctl status snap.microk8s.daemon-kubelet.service 
. . .
# ● snap.microk8s.daemon-kubelet.service - Service for snap application microk8s.daemon-kubelet
#    Loaded: loaded (/etc/systemd/system/snap.microk8s.daemon-kubelet.service; enabled; vendor preset: enabled)
#    Active: active (running) since Wed 2020-05-06 21:17:47 UTC; 1 weeks 6 days ago
#  Main PID: 20171 (kubelet)
#     Tasks: 18 (limit: 4915)
#    CGroup: /system.slice/snap.microk8s.daemon-kubelet.service
#            └─20171 /snap/microk8s/1378/kubelet --kubeconfig=/var/snap/microk8s/1378/credentials/kubelet.config --cert-dir=/var/snap/microk8s/1378/certs # --client-ca-file=/var/snap/microk8s/1378/certs/ca
# 
# May 06 21:18:03 micro microk8s.daemon-kubelet[20171]: E0506 21:18:03.768849   20171 kuberuntime_manager.go:895] Failed to stop sandbox {"containerd" # "b63ab211094478b28d87e5820237b3a6407abfd1173a7fbbd8f
# May 06 21:18:03 micro microk8s.daemon-kubelet[20171]: E0506 21:18:03.768910   20171 kuberuntime_manager.go:674] killPodWithSyncResult failed: failed to "KillPodSandbox" # for "837e5d41-b7b2-4200-8ec9-426
# May 06 21:18:03 micro microk8s.daemon-kubelet[20171]: E0506 21:18:03.768940   20171 pod_workers.go:191] Error syncing pod 837e5d41-b7b2-4200-8ec9-426d48fe4263 # ("hostpath-provisioner-75fdc8fccd-dcjnr_ku
# May 06 21:18:04 micro microk8s.daemon-kubelet[20171]: E0506 21:18:04.524422   20171 remote_runtime.go:128] StopPodSandbox # "39bcf835671f43dc184beb00621f9f1223ca965101ba4ebbe9ed637383430fb5" from runtime
# May 06 21:18:04 micro microk8s.daemon-kubelet[20171]: E0506 21:18:04.524501   20171 kuberuntime_manager.go:895] Failed to stop sandbox {"containerd" # "39bcf835671f43dc184beb00621f9f1223ca965101ba4ebbe9e
# May 06 21:18:04 micro microk8s.daemon-kubelet[20171]: E0506 21:18:04.524556   20171 kuberuntime_manager.go:674] killPodWithSyncResult failed: failed to "KillPodSandbox" # for "14834bd0-6a08-4762-826a-a32
# May 06 21:18:04 micro microk8s.daemon-kubelet[20171]: E0506 21:18:04.524582   20171 pod_workers.go:191] Error syncing pod 14834bd0-6a08-4762-826a-a32aeaa19c2d # ("coredns-588fd544bf-gpjtm_kube-system(148
# May 06 22:34:01 micro microk8s.daemon-kubelet[20171]: I0506 22:34:01.302655   20171 topology_manager.go:233] [topologymanager] Topology Admit Handler
# May 06 22:34:01 micro microk8s.daemon-kubelet[20171]: I0506 22:34:01.348605   20171 reconciler.go:224] operationExecutor.VerifyControllerAttachedVolume started for # volume "pvc-3dc83c05-a518-495f-a865-e
# May 06 22:34:01 micro microk8s.daemon-kubelet[20171]: I0506 22:34:01.348690   20171 reconciler.go:224] operationExecutor.VerifyControllerAttachedVolume started for # volume "default-token-b9fhv" (UniqueN

```