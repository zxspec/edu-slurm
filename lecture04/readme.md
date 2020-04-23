# k8s

## autocompletion

[documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion)

```
kubectl completion -h
```

## Pod example

```
kubectl create -f my-pod.yaml
kubectl get pods
kubectl delete -f my-pod.yaml


kubectl create -f my-pod-1.yaml
kubectl create -f my-pod-2.yaml
kubectl get pods
kubectl delete --all pods
```

## ReplicaSet example

```
kubectl create -f my-replicaset-3.yaml
kubectl get replicasets

#scale up
kubectl apply -f my-replicaset-5.yaml
kubectl get replicasets

#scale down
kubectl apply -f my-replicaset-3.yaml
kubectl get replicasets

kubectl scale --replicas=2 replicaset web
kubectl describe replicasets web
kubectl get pod --show-labels
```
