---
title: "EKS Fargate"
date: 2022-08-06T07:51:46+05:00
draft: false
toc: true
featured_image: '/images/eks-fargate.png'

---

![Image alt](/images/eks-fargate.png)
# EKS with Fargate
EKS can run containers on EC2 or Fargate `Micro VMs` or both.
In case of fargate only mode, EKS asks "hey fargate can you run containers for me."
With fargate you don't have to manage scaling with Auto Scaling Group like we do in case of EC2. 

Normally when we run EC2, we can see those machines running in EC2 dashboard but fargate is only visible in EKS cluster detail section named `culster>compute>nodes` along with fargate profile..

### Fargate Profile
Fargate profile is an interface to configure fargate on an EKS cluster.
Fargate Profile Template tells us which pods should run on fargate.  Below is the sample template. 



```json
"status": "active",
"subnets":["..."],
"clusterName": "MY_CLUSTER_X1",
"fargateProfileArn": "arn...",
"selectors": [
    {
        "namespace": "default"
    },
    {
        "namespace": "kube-system"
    },
    {
        "labels":{
            "foo": "bar",

        },
        "namespace": "my-namespace"
    }
],
"fargateProfileName": "FargateProfileCatchAll",
"podExecutionRole": "arn...",
"createdAt": 343491919.234


```

#### selectors
Notice the `selectors` or we can call pod selectors. its a combination of namespaces and labels which allows EKS to catch pod deployment. Using namespaces and labels we can categories our apps like X1-Sync-Service, X1-Web-API and labels like dev,test,prod

#### subnets
Subnets to pick for the pod deployment or in other way we are choosing availability zones(AZs) where our pods will be placed.

##### podExecutionRole
IAM role to be associated to the kubelet. why kubelet needs IAM role. Just like we assign role to EC2 so it can access s3 etc. 
similarly we assign kubelet an IAM role so 
1. kubelet can pull images from ECR Elastic container registry
2. kubelet can push logs to cloudwatch

As we know pods run inside nodes/machines. similarly when EKS have to run the pod inside fargate micro-vms it will check fargate profile matches with pod labels and namespace.

Here is the step by step process through which pod lands inside fargate.

1. Let say we have pod with namespace=test and label=project1 and we want to launch it.
2.  webhooks `Mutating/validating` will match namespace and label with fargate profile. 
3. if pod namespace and labels matches with fargate selectors then kubernetes/eks scheduler will schedule the pod inside VM.


### Fargate Scheduler
Both kubernetes scheduler and fargate scheduler runs alongside. their purpose is the same like which pods to place on which nodes.


### First install eksctl 
eksctl is tool to launch eks cluster very easily. its an official aws tool.
```shell
#download binary to tmp folder
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

#move binary from tmp to bin
sudo mv /tmp/eksctl /usr/local/bin

#check is installation successful and its version
eksctl version
```
Next we will create cluster with fargate.

### Creating an EKS cluster with fargate using eksctl

```shell
eksctl create cluster --name eksFargateDemo --fargate --region us-east-2
```
above naive command will lot of things like creating cluster via cloudformation, new cluster,  VPC, subnets, ENI and others lot of stuff.

we can use existing VPC by specifying CIDR and subnets stuff but lets keep it simple.

Next if your command is successful. then few things needs to be noticed. 
1. Goto AWS console and see EKS cluster section where `eksFargateDemo` will be listed there. 
2. Also eksctl will save config file inside your home directory with .kube folder. you can verify by running this command. `cat ~/.kube/config` 



```shell
#list all pods
kubectl get pods -A

NAME                                                    STATUS   ROLES    AGE   VERSION
fargate-ip-192-168-107-162.us-east-2.compute.internal   Ready    <none>   13h   v1.22.6-eks-14c7a48
fargate-ip-192-168-98-182.us-east-2.compute.internal    Ready    <none>   13h   v1.22.6-eks-14c7a48
```

```shell
#list all fargate nodes
kubectl get nodes

NAME                                                    STATUS   ROLES    AGE   VERSION
fargate-ip-192-168-107-162.us-east-2.compute.internal   Ready    <none>   13h   v1.22.6-eks-14c7a48
fargate-ip-192-168-98-182.us-east-2.compute.internal    Ready    <none>   13h   v1.22.6-eks-14c7a48
```

Notice we have two pods and two fargate nodes are running so each node will have one pod right now.

Next we can deploy some sample app defined by `myweb.yml`

```
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: myweb
spec: 
  replicas: 1
  selector: 
    matchLabels: 
      run: myweb
  template: 
    metadata: 
      labels: 
        run: myweb
    spec: 
      containers: 
        - 
          image: nginx
          name: my-nginx
          ports: 
            - 
              containerPort: 80


```


```bash
#deploy myweb 
kubectl apply -f myweb.yml

deployment.apps/myweb created
```

```shell
#list all pods again
kubectl get pods -A

NAMESPACE     NAME                      READY   STATUS    RESTARTS   AGE
default       myweb-67bb7495d8-hfzmx    1/1     Running   0          4m23s
kube-system   coredns-9777f8b68-2q2cw   1/1     Running   0          14h
kube-system   coredns-9777f8b68-cqp5n   1/1     Running   0          14h
```

```shell
kubectl get nodes
NAME                                                    STATUS   ROLES    AGE    VERSION
fargate-ip-192-168-107-162.us-east-2.compute.internal   Ready    <none>   14h    v1.22.6-eks-14c7a48
fargate-ip-192-168-150-220.us-east-2.compute.internal   Ready    <none>   4m9s   v1.22.6-eks-14c7a48
fargate-ip-192-168-98-182.us-east-2.compute.internal    Ready    <none>   14h    v1.22.6-eks-14c7a48
```

#### Why EKS
EKS install,maintain and operates kubernetes core feature called control plane or nodes. We will have to do lot of work if we setup kubernetes on EC2. 
EKS integrated with ECR, ELB, IAM and VPC.

#### Why EKS and fargate
EKS have almost same benefits which kubernetes have. 

1. We only care about app spec not infrastructure nodes.
2. automatic cluster scaling. 
3. automatic host patching we only take care of things inside container.
4. Because containers are isolated inside their own pods.
5. we just update cluster and relaunches pods.
6. EFS for stateful apps, built-in logging and load-balancing.

### Fargate Limitations
- Full privileged access is blocked and hiding linux capabilities. 
- No support for general purpose DaemonSets
- No GPUs
- No Amazon EBS Support

