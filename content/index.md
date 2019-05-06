# Kubernetes for Developers



## Slides  
https://yehiyam.github.io/kubernetes-course/#/



## Samples  
https://github.com/yehiyam/kubernetes-course-samples



## Agenda
- Docker <!-- .element: class="fragment" -->
- Kubernetes - basic <!-- .element: class="fragment" -->
- Environments <!-- .element: class="fragment" -->
- Lunch <!-- .element: class="fragment" -->
- Practice - basic kubernetes <!-- .element: class="fragment" -->
- Advanced topics in Kubernetes <!-- .element: class="fragment" -->
- Good practices <!-- .element: class="fragment" -->
- Practice <!-- .element: class="fragment" --> 
 



### What does Kubernetes actually do and why use it?  
vendor-agnostic cluster and container management tool
open-sourced by Google in 2014  
platform for automating deployment, scaling, and operations of application containers across clusters of hosts  



# Open Source
Note:
Created by google in 2014
From borg - internal google orchastration tool



# CNCF
Donated to the Cloud Native Computing Foundation
![pr](./content/images/CNCF-projects.png) <!-- .element: class="fragment" height="50%" width="50%" --> 



# 40k PR/year    
![pr](./content/images/k8spr.png) <!-- .element: class="fragment" --> 



# 60% +
Of enterprises run kubernetes



## Let's backtrack
platform for automating deployment, scaling, and operations of application __containers__ across clusters of hosts
### What are containers? <!-- .element: class="fragment" --> 



### What is a container
- a lightweight OS-level virtualization method <!-- .element: class="fragment" --> 
- stand-alone piece of executable software <!-- .element: class="fragment" --> 
- NOT a virtual machine <!-- .element: class="fragment" --> 


### This is a VM
![pr](./content/images/vm.png ) <!-- .element: height="50%" width="50%"--> 
- The entire OS 
- Binaries and libraries
- Your app


### This is a container
![pr](./content/images/containers.png ) <!-- .element: height="50%" width="50%"--> 
- Containers include the app and dependencies 
- Shares the kernel and drivers with other applications
- Runs as an isolated process on the host OS



## Linux Containers
- Old idea - circa 2000
- Integrated into the Linux kernel around 2008 and became stable in 2013
- Allow running multiple isolated processes on the same host
- Allows resource constraints - CPU, memory, I/O, etc...


### Based on Linux functionality
| Chroot        | Cgroups           | Namespaces  |
| ------------- |:-------------:| -----:|
| Defines the root of the filesystem for the process | Kernel Control Groups allow grouping and partitioning of processes | allow isolation of processes resources like network, users, etc. |


### Using it is hard
Lots of hand wiring of resources
![pr](./content/images/imposiblle.gif) <!-- .element: class="fragment" --> 



### To make things easy
Several projects has created "engines" to abstract all of the hard stuff  

- Docker - we'll talk about it later<!-- .element: class="fragment" --> 
- Rkt - from CoreOS. The first challenger to Docker. Emphasis on security<!-- .element: class="fragment" --> 
