---
title: "Kubernetes Basics"
date: 2022-08-03T21:40:14+05:00
draft: false
featured_image: '/images/kubernetes-logo.png'
---

We will see basic information about kubernetes, architecture and its compoentes and Features 

### kubernetes Architecture

Control plane have following 4 components: CASE
1. Controller Manager -- keeps tracks of whats happening i.e restarting
2. API Server -- scripts, UI, API, CLI
3. Scheduler -- ensures pods replacement. on which node pod to place
4. etcd database -- kubernetes backing store


### Virtual Network
creates one unified machine. its how master and child nodes communicate.
Inside node each pod have its own ip.


## Features
 1. Automated rollouts and rollback
 2. Service Discovery and load balancing
 3. Storage orchestration
 4. Secret and configuration management
 5. Automated bin packing
 6. Batch execution
 7. IPv4/IPv6 dual-stack
 8. Horizontal Scaling
 9. Self Healing
 
### Node & Pods	
Node have multiple pods
pod mostly have single container
- pod is abstraction of container so that any kind of container can run inside pod.
- smallest unit in kubernetes
- usually one application per pod or container
- each pod gets its own container
- pods are ephemeral. avoid using database or anything you want to be consistent.


### Service
1. Internal (Default) (something private like database)
2. External (something needs to be exposed like public api)

### Kubernetes few components and facts
1. Kubelet is primary node agent
2. Control plane have multiple nodes
3. Each node have its own kubelet
4. Each node can have multiple containers/pods


##### Pod
Pod is abstraction of container so that any kind of container can run inside pod.

##### Service
A group of pods declared as network service with one DNS. its service responsibility to load balance traffic in the pods.


##### ConfigMap
A way to store environment variables so pods can access them. credentials should be saved in secretes not configMap

##### Secret
Mechanism to store credentials, API keys etc. something like AWS Secrets manager or Ansible vault.


##### Ingress
Manages connections comming from outside cluster and forwarding to the services inside.

##### Deployment
Deployment provides way to update pods. create new or replace pods and replica set.

##### Stateful set
While deployment and scaling of pods it maintains ordering and uniqueness. 

##### Daemon set
Run copy of pods. When node added/deleted to the cluster pods are added/removed from nodes. Remvoing Daemonset will remove the pods created by the Daemonset.
