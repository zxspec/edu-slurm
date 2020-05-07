## Image - registry:2
```
docker pull registry:2
```

## Storage
```
kubectl create -f local-registry-pvc.yaml
kubectl get pv
. . .
# NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                               STORAGECLASS        REASON   AGE
# pvc-2f9f46ac-44b8-4370-8d57-7e516982b11c   1Gi        RWX            Delete           Bound    default/fileshare                   microk8s-hostpath            4d8h
# pvc-3dc83c05-a518-495f-a865-e30031ded24e   20Gi       RWO            Delete           Bound    default/local-registry              microk8s-hostpath            11m 
#                                                                                                            ^^^  
  
patch pv pvc-3dc83c05-a518-495f-a865-e30031ded24e -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'

kubectl get pv pvc-3dc83c05-a518-495f-a865-e30031ded24e
. . .
# NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS        REASON   AGE
# pvc-3dc83c05-a518-495f-a865-e30031ded24e   20Gi       RWO            Retain           Bound    default/local-registry   microk8s-hostpath            12m
#                                                                       ^^^^
```

```
```