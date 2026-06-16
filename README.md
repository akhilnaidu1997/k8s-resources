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

We have different types of services like clusterIP, NodePort, LoadBalancer, Headless Service
```

## what is replicaset and how it works in the backgorund?
```
Replicaset ensures that the desired no of pod replicas are running at any given point of time.
Since pods are ephemeral and can go down at any time, replicaset ensures spinning up new pod.
We have replicaset controller in controller manager which watches the API server.
And compares the desired state with the actual state and if replica count not matches then it spins up new pod objects.
scheduler finds the suitable node for the pod.
Kubelete starts the container invoking container runtime.
```

## Liveness and Readiness Probe:
```
K8s by default checks only if the process is running or not but it wont checks if the app is healthy or not.
Sometimes due to memory leaks and app deadlocks the app may be running but not healthy.
During this we should not route traffic to the pods.
Hence we need to define livness and readiness probes.

liveness : determines when to restart the container.
readiness : determines whether to route the traffic to the pod or not.

kubelet is responsible for performing the probes on containers.
if liveness fails then kubelet tells container runtime to restart container.
if readiness fails then pod is removed from the service endpoints and traffic wont be routed tp that pods.

Here we can define probes using 3 methods : httpGet, exec, tcp socket
```

## Daemonset:
```
Daemonset ensures that a copy of pod replica runs on all eligible nodes.
If a daemonset pod goes down then it spins up new pod we have daemonset controller in  controller manager which is responsible for spinning up pods.
Even if new node is added then daemonset controller notices the node and spins up the new pod here.
Mostly used with the agents like fluentd, fluentbit, filebeat, node exporter for monitoring and logging. These agents collects the logs and metrics from nodes and push to central location like ELK and prometheus, cloudwatch for node health monitoring.
These pods will have read only access to hostpath to collect logs from /var/log folder.
```

## EBS vs EFS
```
EBS ( Elastic block storage)
EBS volumes should be in same AZ as that of instance.
EBS volumes can be mounted to only one ec2 instance
volumes cant grow as per usage but can be resized.
it has low latency and high IOPS
Mostly used to store OS, Databases

EFS is a managed service and its serverless.
Though we can achive this using ec2 instance by setting up the NFS packages but we are responsible for patching, scaling, pay data transfer costs and pay per hour ec2 instance costs.
But here aws provides EFS which is highly secure, reliable, 99.9999 11 9's durability and 99.99 high availablity offered by aws.
It works on NFS v4 protocol.
We can also trasnsition data stored from one storage class to another automatically.
can be mounted across multiple ec2 instances for shared access.
```

## HEadless Service
```
Headless service will not have clusterIp and no load balancing.
Querying headless service DNS gives all Pod Ips and we can establish communication with the required pod.
Kube proxy doesnot managed the headless service.
For provisioning headless svc we can define spec.clusterIP: None then we can get the headless svc created.
Used for stateful applications where requires peer discovery for data replication.
```

## statefulset
```
Statefulset pods are used for stateful applications like DB's.
it maintains pod identity creates pods in orderly manner and deletes in reverse order.
if a pod goes down then it creates another pod with same and binds to same volumes.
We need headless svc, cluster IP svc and volumes.
Each pod can have its own volumes where we use storage classes, volume claim templates.
```

## What happens when pod crashes
```
When a pod crashes, here kubelet updates the node status to the API server periodically.
Replicaset controller in controller manager watches for these status, compares the desired state with the actual pod count.
If it not matches then creates a pod object. Now scheduler finds the suitable node for the pods based on the taints, affinity rules and resource availability.
kubelet invokes the container runtime to start the container and this way it maintains the desired no of replicas all time.
```

## What exactly happens when a Kubernetes pod gets OOMKilled?
```
when a kubernetes pod gets OOM killed, here applications reaches its memory consumption limits.
Now cgroups detects the limits exceeding and kernel triggers the OOM killer which send SIGKILL to the container.
Container gets exited with the exit code 137. Kubelet detects this and tries to restart the container with help of container runtime.
Now this gets repeated failures and eventually pod goes into crashloopbackoff.
This occurs due to multiple issues like pods require more memory requests while starting up pod and insuffuicient memory limits and inf memory leak issues.

kubectl get pods
kubectl describe pod <pod name>
kubectl logs <pod name>

Short term fix:
increase the no of pod replicas
increase the limits
restart the affected pods.

Longterm fix:
Analyse the logs to understand why memory is keeps consuming due to memory leaks issue
And work with the developers to fix it
change memory limits if requires
```

```
Cluster-level scope:
Most components in kube-system (like kube-dns/CoreDNS, kube-proxy, kube-controller-manager, kube-scheduler, etc.) operate across the entire cluster, not just within the kube-system namespace.

Why they are in a namespace:
Even though they are cluster-scoped in function, Kubernetes still organizes their Pods, ConfigMaps, and Services into the kube-system namespace for logical grouping and isolation.
This helps:

Avoid accidental modification by users.
Apply RBAC rules more easily.
Keep system resources separate from application workloads.
Examples:

Component	Scope	Runs in kube-system?
kube-apiserver	Cluster-level	Yes
kube-controller-manager	Cluster-level	Yes
kube-scheduler	Cluster-level	Yes
etcd	Cluster-level	Yes
CoreDNS	Cluster-level	Yes
kube-proxy	Cluster-level	Yes
```
