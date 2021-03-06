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
* From Canonical - the makers of Ubuntu
* Very easy to install
* Installs as _snap_ 

```
$ sudo snap install microk8s --classic
```



### Docker Desktop
* From Docker
* Installs Docker and Kubernetes in a VM



### Cloud Providers
* All major cloud providers has a managed (paas) Kubernetes offering.
 * GKE - Google
 * EKS - Amazon AWS
 * AKS - Microsoft Azure
 * Openshift, Pivotal, IBM, and many more
* Very easy to set up - Usually one click


### On Premise
* Several of the paas Kubernetes providers also support on-prem installation
* Even in a disconnected environment
* We use [Kubespray](https://github.com/kubernetes-sigs/kubespray)
 * Ansible scripts
 * Together with bash scripts


### Playgroud
* [Katacoda](https://www.katacoda.com/) is an interactive learning and training platform for software developers
* Provide web (browser) access to hosted environments that can be customized
* Provides curses and tutorials for Docker, Kubernetes and many more



### kubectl command line
Official CLI tool
```
$ kubectl version
$ kubectl --help
```
Has autocomplete
```
$ echo 'source <(kubectl completion bash)' >>~/.bashrc
```
It is configured with the file  
```
~/.kube/config
```
> minikube, microk8s will configure it for us


### kubectl
Can be configured for multiple contexts
```
kubectl config get-context
```
Switch to a certain context:
```
$ kubectl config use-context minikube
```


### kubectl get
```
$ kubectl get pods
$ kubectl get deployments
```
Filter with labels
```
$ kubectl get pods -l key=value
```
Output format
``` 
$ kubectl get deployment -o wide/json/yaml
```
Watch for changes
```
$ kubectl get po -l type=worker -w
```


### kubectl create
Create from yaml file
```
$ kubectl create -f path/to/yaml
$ kubectl apply -f path/to/yaml
```
> apply also allows to update the object


### kubectl logs
```
$ kubectl logs podName 
```
Follow logs
```
$ kubectl logs podName -f
```


### kubectl exec
Execute command in a running container
```
$ kubectl exec -it podName command 
```
Example - run bash shell in a container
```
$ kubectl exec -it podName /bin/bash 
```


### kubectl port-forward
Forward TCP traffic from the pod to the local machine
```shell
$ kubectl port-forward podName localPort:podPort
```
> Can be used for remote debugging (Later)




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


<!-- .slide: data-transition="none" -->
### Resources
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
    <u>resources:</u>
      limits:
        memory: "128Mi"
        cpu: "500m"
      requests:
        memory: "64Mi"
        cpu: "200m"
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


### Service Type ClusterIP
* Not exposed outside cluster.
* Suitable for communication within cluster 
>Example: A frontend application talking to a backend without exposing the
backend outside the cluster.


### Service Type: NodePort
* Exposed through a port on each of the nodes in the cluster
* Port range 30000-32767
> Example: Allow a physical Load Balancer to access service


### Service Type: LoadBalancer
* Cloud specific implementation


### Services Type: Headless service
* No endpoint is created
* Can be used to return a list of pod IP's



### Configuration
* Applications expect configuration from
 * Configuration files
 * Command line arguments
 * Environment variables


### Configuration
* Configuration is always decoupled from applications
 * INI
 * XML
 * JSON
 * Custom Format
* Container Images shouldn't hold application configuration
 * Essential for keeping containerized applications portable


### ConfigMaps
* Kubernetes objects for injecting containers with configuration data
* ConfigMaps keep containers agnostic of Kubernetes
* They can be used to store fine-grained or coarse-grained configuration
 * Individual properties
 * Entire configuration file
 * JSON files
* ConfigMaps hold configuration in Key-Value pairs accessible to Pods


### ConfigMap Spec
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```


<!-- .slide: data-transition="none" -->
### ConfigMap Spec
<pre>
apiVersion: v1
<u>kind: ConfigMap</u>
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
</pre>


<!-- .slide: data-transition="none" -->
### ConfigMap Spec
<pre>
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
<u>data:</u>
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
</pre>


### Use it in pods as Env
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: SPECIAL_LEVEL
```


### Use it in pods as Files
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: front-end
      image: nginx
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
```



### Secrets
* Secrets keep user data like config maps
* Limited to small amount of data like tokens and passwords
* Secrets are encrypted between Api Server to the Kubelet, but
 * Kept as clear text in ectd



## Lets practice
https://www.katacoda.com/courses/kubernetes/creating-kubernetes-yaml-definitions



## Networking


Kubernetes network needs to solve several distinct networking problems


### Container to Container
Solved by the Pod concept and `localhost`


### Pod-to-Service communications
Solved with _kube-proxy_


### Pod to Pod
Kubernetes requirements from pod networking:
* Every pod has a unique IP address
 * No need to map ports
 * Almost like VM
* All pods can communicate without NAT
* Agents on a node can communicate with all pods on that node


<!-- .slide: data-transition="none" -->
### Lets start with docker
![net1](./content/images/net1.png)


<!-- .slide: data-transition="none" -->
### Add another
![net1](./content/images/net2.png)


<!-- .slide: data-transition="none" -->
### Pod with two containers
![net1](./content/images/net3.png)


<!-- .slide: data-transition="none" -->
### Actually there is a third container
![net1](./content/images/net4.png)


<!-- .slide: data-transition="none" -->
### And if we have two nodes
![net1](./content/images/net5.png)<!-- .element width="70%" height="70%"-->


<!-- .slide: data-transition="none" -->
### But this is wrong
![net1](./content/images/net5wrong.png)<!-- .element width="70%" height="70%"-->


<!-- .slide: data-transition="none" -->
### The kubernetes way
![net1](./content/images/net6.png)<!-- .element width="70%" height="70%"-->


But the Pod IP's are not routable between hosts


### Overlay/Underlay Networks
* Provide a mechanism to get packets from one host to another
* May operate on Layer 2 or Layer 3
* Many options


Cloud providers usually use smart networking tricks (Underlay)


Otherwise there is a need to wrap packets and route them to the correct host (VxLAN)


* Flannel
* Project Calico
* Cilium
* Contiv



## Advanced Objects
* Daemon set
* Job
* Cronjob
* Ingress
* Storage


### Daemon Set
* A DaemonSet ensures that all (or some) Nodes run a copy of a Pod
* Some typical uses of a DaemonSet are:
 * running a logs collection daemon on every node, such as fluentd
 * running a node monitoring daemon on every node


### Job
* A Job creates one or more Pods and ensures that a specified number of them successfully terminate
* If a Pod failes, the Job controller will start a new one
 * Until the required number of successful completions
 * Or the maximum number of retries


### Job Spec
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```


### CronJob
* A Cron Job creates Jobs on a time-based schedule.
* Similar to crontab
 * Runs a job periodically
 * Schedule is written in Cron format


### CronJob Spec
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```


<!-- .slide: data-transition="none" -->
### CronJob Spec
<pre>
apiVersion: batch/v1beta1
<u>kind: CronJob</u>
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
</pre>


<!-- .slide: data-transition="none" -->
### CronJob Spec
<pre>
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  <u>schedule: "*/1 * * * *"</u>
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
</pre>


<!-- .slide: data-transition="none" -->
### CronJob Spec
<pre>
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  <u>jobTemplate:</u>
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
</pre>



### Persistent Storage (Volumes)
* As we saw before, Pods are ephemeral and stateless
* Volumes brings persistance to Pods
* All containers in a Pod can access the volume
* Volumes are associated with the lifecycle of Pod


<!-- .slide: data-position="top" --> 
### Pods and Volumes
![podsAndVolumes](./content/images/podsAndVolumes.png)


### Types of Volumes
Host Based|Block|Network
----------|:---:|-------
EmptyDir|Amazon EBS|NFS
HostPath|GCE Persistent Disk|Ceph
        |Azure Disk|Gluster


### Persistent Volumes & Claims
* PersistentVolume (PV)
 * Networked storage in the cluster pre-provisioned by an administrator.
* PersistentVolumeClaim (PVC)
 * Storage resource requested by a user.
* StorageClass
 * Types of supported storage profiles offered by administrators


### Storage Provisioning Workflow
![storage](./content/images/storage.png)



### Exposing Services and Ingress
* Services expose application running in pods
* Services provide a stable endpoint for clients
* Ingress adds intelligent routing on top of services


### Ingress 
* Ingress is composed of two parts
* Ingress Resource
 * Defines (configures) routing conditions to a service
* Ingress Controller
 * A controller (Pod) that watches for resources and updates its rules accordingly
 * Common Ingress Controllers: Nginx, HAProxy, Envoy and more


### Ingress Resource
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: simulator
  namespace: default
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: simulator
          servicePort: 9050
        path: /hkube/simulator
```


<!-- .slide: data-transition="none" -->
### Ingress Resource
<pre>
apiVersion: extensions/v1beta1
<u>kind: Ingress</u>
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: simulator
  namespace: default
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: simulator
          servicePort: 9050
        path: /hkube/simulator
</pre>


<!-- .slide: data-transition="none" -->
### Ingress Resource
<pre>
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: simulator
  namespace: default
spec:
  rules:
  - http:
      paths:
      <u>- backend:</u>
          serviceName: simulator
          servicePort: 9050
        path: /hkube/simulator
</pre>


<!-- .slide: data-transition="none" -->
### Ingress Resource
<pre>
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: simulator
  namespace: default
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: simulator
          servicePort: 9050
        <u>path: /hkube/simulator</u>
</pre>


### Ingress
![ingress](./content/images/ingress.png)



## Good practices


### Kubernetes is really flexible


### But you might shoot yourself in the foot


### One process per container


### Don't restart on failure - Crash cleanly


### Log to stdout and stderr


### Use plenty of descriptive labels


<pre>
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: rss-site
spec:
  replicas: 2
  template:
    metadata:
      labels:
        <u>app: web</u>
        <u>revision: 3</u>
        <u>experimental: true</u>
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


![1](./content/images/kubernetes-best-practices-32-638.jpg)


<!-- .slide: data-transition="none" -->
![1](./content/images/kubernetes-best-practices-33-638.jpg)


<!-- .slide: data-transition="none" -->
![1](./content/images/kubernetes-best-practices-34-638.jpg)


<!-- .slide: data-transition="none" -->
![1](./content/images/kubernetes-best-practices-35-638.jpg)


<!-- .slide: data-transition="none" -->
![1](./content/images/kubernetes-best-practices-36-638.jpg)


### Use sidecar containers for proxies, loggers, etc


<!-- .slide: data-transition="none" -->
![1](./content/images/kubernetes-best-practices-39-638.jpg)


### Init Containers
* Init Containers are exactly like regular Containers, except:
 * They always run to completion.
 * Each one must complete successfully before the next one is started.


### Example usage:
Wait for a service to be created with a shell command like:
```
for i in {1..100};
do 
  sleep 1
  if dig myservice; then exit 0; fi
done
exit 1
```


Register this Pod with a remote server


Wait for some time before starting the app Container with a command like
```
sleep 60
```


<!-- .slide: data-transition="none" -->
### Init Container Spec
In Pod Spec
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  <u>initContainers:</u>
  - name: init-myservice
    image: busybox:1.28
    command: ['sh','-c', 'until nslookup mysvc; do sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do sleep 2; done;']
</pre>


<!-- .slide: data-transition="none" -->
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  <u>initContainers:</u>
  - name: init-myservice
    image: busybox:1.28
    command: ['sh','-c', 'until nslookup mysvc; do sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do sleep 2; done;']
</pre>


<!-- .slide: data-transition="none" -->
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  <u>- name: init-myservice</u>
    image: busybox:1.28
    command: ['sh','-c', 'until nslookup mysvc; do sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do sleep 2; done;']
</pre>


<!-- .slide: data-transition="none" -->
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    <u>image: busybox:1.28</u>
    command: ['sh','-c', 'until nslookup mysvc; do sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do sleep 2; done;']
</pre>


<!-- .slide: data-transition="none" -->
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    <u>command: ['sh','-c', 'until nslookup mysvc; do sleep 2; done;']</u>
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do sleep 2; done;']
</pre>


<!-- .slide: data-transition="none" -->
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh','-c', 'until nslookup mysvc; do sleep 2; done;']
  <u>- name: init-mydb</u>
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do sleep 2; done;']
</pre>


### Container health checks
* Kubernetes provides two health checks for containers
 * Rediness: Is the app ready 
 * Liveness: Is the app still running

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```


### Node Selector
* Allows node selection constraint
* A map of key-value pairs
 * All of the pairs must match the node labels


### Attach labels to nodes
```shell
$ kubectl label nodes <node-name> <label-key>=<label-value>
```


### Add nodeSelector to Pod
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  <u>nodeSelector:</u>
    disktype: ssd
</pre>


### Pod Affinity/Anti-affinity
* Allows grouping or separating Pods across nodes
* Example usage:
 * Schedule two pods on the same node for data locality
 * Don't schedule two replicas of database pods to the same node


### Namespaces


Kubernetes supports multiple virtual clusters backed by the same physical cluster


These virtual clusters are called namespaces


* Namespaces provide a scope for names
* Names of resources need to be unique within a namespace, but not across namespaces


For example  
kube-system namespace that is created by Kubernetes


### Helpful tools
* Telepresence
* Remote Debugging
* Visual Studio Code Kubernetes Tools


### Telepresence
https://www.telepresence.io  
* Open source tool
* Lets you run a single service locally, while connecting that service to a remote Kubernetes cluster
* Deploys a two-way network proxy in a pod running in your Kubernetes cluster


### Remote Debugging
Using port forwarding to pods in the cluster to debug
Nodejs 
```shell
$ kubectl exec -it pod-name -- bash -c 'kill -s USR1 1'
$ kubectl port-forward pod-name 9229:9229
# open chrome and connect
```


### Visual Studio Code Kubernetes Tools
* View cluster resources
* Edit/Create resources  

Remote debug a pod in kubernetes<!-- .element: class="fragment" -->