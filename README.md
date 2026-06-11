## what is Namespace
```
Namespace is a logical isolation within the k8s cluster. its like a virtual cluster within the k8s cluster.
If we want to use same cluster for multiple projects then we can have project specific namespaces and can deploy apps with respect to it.
We can also have diff environments in the same cluster.
We can control the resource quotas at the namespace level. This way we can prevent one particular project from consuming all cluster resources

kubectl get ns
kubectl create ns <name>
```

## what is pod?
```
Pod is the smallest deployable unit in k8s. Each pod can have one or more containers.
containers within the pod can communicate using localhost.
containers within the pod can share same storage and networking.
These are abstracted away from the containers and handled at the pod level.
Each pod can have its own unique IP address which is attached by VPC CNI
Pods are ephemeral in  nature and can go down at any time.

kubectl get pods
kubectl get pods -n <namespace>
kubectl describe pod <name>
```

## Where does the Pods get IPs from?
```
Here VPC CNI is responsible for assigning IPs to pods. But CNI gets these IPs from the worker nodes, worker nodes will have secondary IPs and these are used for pods.
Based on the instance type that we choose like m5.xlarge here it will have 4 ENIs and each ENI can have 15 IPs.
Total 60 IPs and usable are 58 means 58 pods can be deployed here.
For a instance based on the type we choose we get primary and secondary IPs.
```

## Multi containers:
```
As we know that pods can have more than one container.
One container can have the main app and other can have monitoring and logging agents.
Lets say I have deployed logging agent like fluentd and pushing logs to the ELK.
multi containers are used for sidecar and proxy. 

kubectl exec -it <name> -c <container-name> -- bash   --> here this command is used to login to specific container in a pod.
```

## Labels and Annotations:
```
Labels are key value pairs that we define to select internal K8s resources.
used for grouping and selecting. selectors uses labels to select pods.
We have service selectors and deployment selectors. And labels do not allow special characters to define and have limited character length of 64.

# kubectl filtering using labels
kubectl get pods -l app=myapp
kubectl get pods -l env=production,app=myapp

Annotations are also key value paris that we define to add metadata to pods.
These are used for selecting the resources external to cluster it can be aws resources.

annotations:
  kubernetes.io/ingress.class: alb
  alb.ingress.kubernetes.io/scheme: internet-facing
  alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:...
```

## Resource management:
```
Advantage with the containers is the dynamic resource utilisation. Based on the traffic containers may end up using entire host resources.
Sometimes due to some issue in the code or memory leaks resources keeps consuming and not released even though there is no much traffic. which affects other containers and host may go down hence it is recommended to define the resources request and limits.

resources:
    requests:
        cpu: 
        memory:
    limits:
        cpu:
        memory:

This can vary from service to service.
```

## configmaps and secret

## Why do we need services?
```
Pods are ephemeral in nature means short lived and can go down at any time.
Each can have its own unique IP address and it changes for every pod restart.
Also we will have multiple pod replicas for the same service for high availability.
We need services for service discovery and load balancing.
This provides stable IP and DNS name for the group of pods and traffic will be routed to the healthy pods.

We have different types of services like clusterIP, NodePort, LoadBalancer
```
