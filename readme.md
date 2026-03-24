## K8S resources ##

## Namespace ##
- Namespace is logical isolation within the cluster. Its like virtual cluster within the cluster.
- when we want to run multiple environments within the cluster then we can make use of namespaces.
- we can restrict the access at the namespace level

```
kubectl get ns
kubectl create ns roboshop
kubectl apply -f namespace.yaml
```

