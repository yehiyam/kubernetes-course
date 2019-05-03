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


### How it all fits together
![pr](./content/images/k8sArch.png) 



### Addons
Usual Kubernetes installations include several addons
* DNS
 * A DNS server that works in conjunction with the system DNS
 * Provides name resolution for services and pods
* Cluster-level Logging - e.g. elasticsearch+kibana
* Dashboard



## Deployments
* Kubernetes can run on various platforms
 * Laptop, desktop
 * VM in the cloud
 * Rack of bare metal servers
* Also in various configurations
 * Single node
 * Single master and multiple nodes
 * HA masters and nodes
* The effort required to set up a stable cluster varies


## Local Deployment
There are several options to install a local Kubernetes instance
* minikube - Linux, Mac and Windows
* microK8s - Linux only
* Docker Desktop - Mac and Windows 


### Minikube
* Official from Kubernetes
* Runs Kubernetes in a VM on the local machine

```
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
$ sudo cp minikube /usr/local/bin && rm minikube
```
```
$ minikube start
```


### MicroK8s
* From Canonical 
* Very easy to install
* Installs as _snap_ 

```
$ sudo snap install microk8s --classic
```


### Docker Desktop
* From Docker
* Installs Docker and Kubernetes in a VM



## API Objects
* The user interacts with kubernetes by setting, modifying and deleting _API Objects_ 
* Examples of objects:
 * Pod
 * Service
 * Deployment
* Objects are persisted in Kubernetes
* Each object has a **spec** and **status**


### How it Works
![pr](./content/images/k8sSimple.png) 


### Breaking it Down
Lets create a pod <!-- .element: style="text-align:left;"-->
* User sends _API Object_ to API Server
* API Server stores it in etcd
* Scheduler schedules the pod to a node
* Kubelet on the node creates the containers/network/mounts for the pod
* Docker pulls the image from the registry and runs the container


### Pods
* Pods are logical wrappers for containers
* All containers in the same pod will:
 * Run on the same host
 * Share the same network
* Usually there will be only one container per pod
* Extra containers are called sidecars


### Pods
* Pods are ephemeral.
 * They won't survive scheduling failures, node failures, or other evictions, such as due to lack of resources
 * The app should be able to recover from  


### Sidetrack - YAML
* A super-set of JSON
* Uses spaces **(not tabs)** instead of {}
* Supports lists and maps


### YAML
Maps
```yaml
apiVersion: v1
kind: Pod
```
similar to JSON
```json
{
    "apiVersion": "v1",
    "kind": "Pod"
}
```


### YAML
Maps of Maps
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rss-site
  labels:
    app: web
```


### YAML
Lists
```yaml
args:
  - sleep
  - "1000"
  - message
  - "Bring back Firefly!"
```


### YAML
Comments
```yaml
apiVersion: v1
kind: Pod
# this is a comment
```


<!-- .slide: data-transition="none" -->
### Pods
<pre>
<u>apiVersion: v1</u>
kind: Pod
metadata:
  name: rss-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: rss-reader
      image: nickchase/rss-php-nginx:v1
      ports:
        - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Pods
<pre>
apiVersion: v1
<u>kind: Pod</u>
metadata:
  name: rss-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: rss-reader
      image: nickchase/rss-php-nginx:v1
      ports:
        - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Pods
<pre>
apiVersion: v1
kind: Pod
<u>metadata:</u>
  <u>name: rss-site</u>
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: rss-reader
      image: nickchase/rss-php-nginx:v1
      ports:
        - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Pods
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: rss-site
  <u>labels:</u>
    <u>app: web</u>
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: rss-reader
      image: nickchase/rss-php-nginx:v1
      ports:
        - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Pods
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: rss-site
  labels:
    app: web
<u>spec:</u>
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: rss-reader
      image: nickchase/rss-php-nginx:v1
      ports:
        - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Pods
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: rss-site
  labels:
    app: web
spec:
  <u>containers:</u>
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: rss-reader
      image: nickchase/rss-php-nginx:v1
      ports:
        - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Pods 
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: rss-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      <u>image: nginx</u>
      ports:
        - containerPort: 80
    - name: rss-reader
      image: nickchase/rss-php-nginx:v1
      ports:
        - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Pods 
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: rss-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: rss-reader
      <u>image: nickchase/rss-php-nginx:v1</u>
      ports:
        - containerPort: 88
</pre>



### Deployment
* As stated before, Pods are not durable, and might come down
* Pods are usually created using a controller, e.g. Deployment, Job, etc.
* Deployment creates and manages a set of pods
 * Ensure the number of replicas 
 * Provides rolling update of versions


### Deployment Spec
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: rss-site
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: front-end
          image: nginx
          ports:
            - containerPort: 80
        - name: rss-reader
          image: nickchase/rss-php-nginx:v1
          ports:
            - containerPort: 88
```


<!-- .slide: data-transition="none" -->
### Deployment spec
<pre>
<u>apiVersion: extensions/v1beta1</u>
kind: Deployment
metadata:
  name: rss-site
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: front-end
          image: nginx
          ports:
            - containerPort: 80
        - name: rss-reader
          image: nickchase/rss-php-nginx:v1
          ports:
            - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Deployment spec
<pre>
apiVersion: extensions/v1beta1
<u>kind: Deployment</u>
metadata:
  name: rss-site
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: front-end
          image: nginx
          ports:
            - containerPort: 80
        - name: rss-reader
          image: nickchase/rss-php-nginx:v1
          ports:
            - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Deployment spec
<pre>
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: rss-site
spec:
  <u>replicas: 2</u>
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: front-end
          image: nginx
          ports:
            - containerPort: 80
        - name: rss-reader
          image: nickchase/rss-php-nginx:v1
          ports:
            - containerPort: 88
</pre>


<!-- .slide: data-transition="none" -->
### Deployment spec
<pre>
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: rss-site
spec:
  replicas: 2
  <u>template:</u>
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: front-end
          image: nginx
          ports:
            - containerPort: 80
        - name: rss-reader
          image: nickchase/rss-php-nginx:v1
          ports:
            - containerPort: 88
</pre>



### Service
* A Service is used to define a logical set of Pods and related policies used to access them
* A Service gives the group of Pods a constant IP and DNS name
* It acts as a load balancer between the Pods
* It is implemented by the _kube-proxy_ instances that configures _iptables_ rules


<!-- .slide: data-position="center" --> 
![pr](./content/images/services-iptables-overview.svg) <!-- .element width="70%" height="70%"-->


### Service Spec
```yaml
apiVersion: v1
kind: Service
metadata:
  name: rss-site
spec:
  type: NodePort
  ports:
  - name: "front-end"
    port: 5001
    targetPort: 80
    nodePort: 31001
  selector:
    app: result
```


<!-- .slide: data-transition="none" -->
### Service Spec
<pre>
apiVersion: v1
<u>kind: Service</u>
metadata:
  name: rss-site
spec:
  type: NodePort
  ports:
  - name: "front-end"
    port: 5001
    targetPort: 80
    nodePort: 31001
  selector:
    app: result
</pre>


<!-- .slide: data-transition="none" -->
### Service Spec
<pre>
apiVersion: v1
kind: Service
metadata:
  name: rss-site
spec:
  <u>type: NodePort</u>
  ports:
  - name: "front-end"
    port: 5001
    targetPort: 80
    nodePort: 31001
  selector:
    app: result
</pre>


<!-- .slide: data-transition="none" -->
### Service Spec
<pre>
apiVersion: v1
kind: Service
metadata:
  name: rss-site
spec:
  type: NodePort
  ports:
  - name: "front-end"
    port: 5001
    targetPort: 80
    nodePort: 31001
  <u>selector:</u>
    app: result
</pre>


