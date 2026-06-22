## PREREQUISITES
```
kubectl
eksctl
aws cli
helm
```
## create aws eks managed node group cluster
```
eksctl create cluster \
 --name=eks-cluster \
 --region=us-east-1 \
 --zones=us-east-1a,us-east-1b \
 --nodegroup-name=standard-nodes \
 --node-type=t3a.medium \
 --nodes=2 \
 --nodes-min=2 \
 --nodes-max=10 \
 --managed
```
## Enable OIDC
Cluster Autoscaler uses IAM Roles for Service Accounts (IRSA)
```
eksctl utils associate-iam-oidc-provider \
--region us-east-1 \
--cluster eks-cluster \
--approve
```
## Create IAM Policy for Autoscaler
```
cat<<EOF > cluster-autoscaler-policy.json
{
 "Version": "2012-10-17",
 "Statement": [
 {
 "Effect": "Allow",
 "Action": [
 "autoscaling:DescribeAutoScalingGroups",
 "autoscaling:DescribeAutoScalingInstances",
 "autoscaling:DescribeLaunchConfigurations",
 "autoscaling:DescribeTags",
 "autoscaling:SetDesiredCapacity",
 "autoscaling:TerminateInstanceInAutoScalingGroup",
 "ec2:DescribeLaunchTemplateVersions"
 ],
 "Resource": "*"
 }
 ]
}
EOF
```
## create IAM policy
```
aws iam create-policy \
--policy-name AmazonEKSClusterAutoscalerPolicy \
--policy-document file://cluster-autoscaler-policy.json
```
## Save the Policy ARN.
```
arn:aws:iam::551322107788:policy/AmazonEKSClusterAutoscalerPolicy
```
## Create Service Account with IAM Role
Replace:
    <ACCOUNT_ID>
    <POLICY_ARN>
```
eksctl create iamserviceaccount \
--cluster eks-cluster \
--namespace kube-system \
--name cluster-autoscaler \
--attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/AmazonEKSClusterAutoscalerPolicy \
--approve \
--override-existing-serviceaccounts
```
## Install Cluster Autoscaler using Helm
You can take the helm chart from artifcathub.io
```
helm repo add cluster-autoscaler https://kubernetes.github.io/autoscaler
helm repo update
helm show values cluster-autoscaler/cluster-autoscaler > autoscaler_values.yaml
```
## Change the below setting in the values:
```
vi autoscaler_values.yaml

clustername
namespace: kube-system
aws region 
rbac.serviceaccount.create=false
rbac.serviceAccount.name=cluster-autoscaler
Come to extraArgs , and enable them with proper settings
extraArgs:
 balance-similar-node-groups: "true"
 skip-nodes-with-system-pods: "false"
 scale-down-unneeded-time: "5m"
 scale-down-delay-after-add: "2m"
 scale-down-utilization-threshold: "0.5"
 Image version - registry.k8s.io/autoscaling/cluster-autoscaler:v1.29.0 - should match the eks version
```

## Deploy a workload:
```
kubectl create deployment nginx --image=nginx
kubectl set resources deployment nginx --requests=cpu=500m,memory=512Mi   
kubectl scale deployment nginx --replicas=20

watch for the pods goes into pending state
kubectl get pods / kubectl get pods -w

watch nodes getting adding
kubectl get nodes / kubectl get nodes -w

After sometime delete the pods and verify again, nodes getting deleted automatically.
```

## Clean Up
Delete the cluster:
```
eksctl delete cluster --name live-eks-cluster --region us-east-1
```