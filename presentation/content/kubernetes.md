# Kubernetes



## Motivation
* Docker lets us use containers easily on our local machine (or any one machine)
* Kubernetes provides a logical abstraction of treating the data-center as a single machine
 * machines
 * storage
 * network
* It allows for deploying, provisioning and self-healing of container-groups (pods)
* Gives you a single interface to deploy containers



## Kubernetes Concepts
* Node - A physical or virtual machine
* Pod - The basic building block. A logical group of one or more containers
* Controllers - Responsible for bringing the cluster to the desired state
* Services - An abstraction over a group of pods  


## Kubernetes Lifecycle
* To change the state of Kubernetes, we use _Kubernetes API_ objects to describe the desired state of the cluster
* The _Kubernetes Control Plane_ works to make the current state of the cluster match the desired state


## Nodes
There are two types of nodes
* master node - runs the _Control Plane_ of the cluster
* worker node - runs user workloads


### Master Node
* a collection of three processes
 * kube-apiserver
 * kube-controller-manager
 * kube-scheduler


### kube-apiserver
* Provides a REST endpoint for the API
* Provides the frontend to the cluster's shared state and control plane
* Designed to scale horizontally


### kube-controller-manager
* Contain the built-in controllers
* In Kubernetes, a controller is a control loop that watches the shared state of the cluster through the apiserver and makes changes attempting to move the current state towards the desired state
* Examples of controllers that ship with Kubernetes today
 * Node Controller: Responsible for noticing and responding when nodes go down
 * Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system
 * Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods)
 * and more


### kube-scheduler
* Responsible for scheduling a pod to a node
* When scheduling, need to look at:
 * individual and collective resource requirements
 * hardware/software/policy constraints
 * affinity and anti-affinity 
 * workloads
 * storage  - data locality
* Basically loops over all unscheduled pods and finds a node for them


### etcd
* Not really part of the _control plane_
* Provides shared storage for the cluster
 * Consistent
 * Highly-available
 * Key value store



### Worker Node
* Node components run on every node
 * kubelet
 * kube-proxy
 * container runtime


### kubelet
* An agent that runs on each node in the cluster
* Runs and stops containers on that node
 * Watches _PodSpec_ and makes sure that the containers described in the are running and healthy
 * Does not manage containers that are created outside of kubernetes


### kube-proxy
* enables service abstraction
* maintains network rules on the host for connection forwarding


### container runtime
* The software responsible for running containers
* Docker is the most widely used
 * also supports containerd, cri-o, rktlet
 * Can be extended through the _Kubernetes Container Runtime Interface_