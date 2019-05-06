## Docker
- Created as open source and released in 2013
- It was the first easy way to use containers



## A little marketing
- Docker is an open platform for developing, shipping, and running applications. <!-- .element: class="fragment" --> 
- Docker is designed to deliver your applications faster. <!-- .element: class="fragment" --> 
- With Docker you can separate your applications from your infrastructure AND treat your infrastructure like a managed application. <!-- .element: class="fragment" --> 
- Docker helps you ship code faster, test faster, deploy faster, and shorten the cycle between writing code and running code.<!-- .element: class="fragment" -->  


### Architecture
* Client server architecture<!-- .element: class="fragment" --> 
* Server daemon executes and admins the processes<!-- .element: class="fragment" --> 
* Uses Linux Kernel namespaces and cgroups<!-- .element: class="fragment" --> 
* Gets commands from API<!-- .element: class="fragment" --> 
* Client - Command line interface (CLI). Issues commands to the daemon<!-- .element: class="fragment" --> 


![pr](./content/images/docker-architecture.svg) 


### Images
* Docker images are read-only templates from which Docker containers are launched.
* Image names have the form <!-- .element: class="fragment" --> 
```
[registry/][user/]name[:tag]
```
* The default for the tag is latest <!-- .element: class="fragment" --> 
 * don't use it <!-- .element: class="fragment" --> 


### Images
* Each image consists of a series of layers.
* Docker makes use of Union (Overlay) file systems to combine these layers into a single image. Union file systems allow files and directories of separate file systems, known as branches, to be transparently overlaid, forming a single coherent file system.<!-- .element: class="fragment" -->


### Images
* Every image starts from a base image.
 * E.g ubuntu, Apache.
* Docker images are then built from these base images using a simple, descriptive set of steps we call instructions.<!-- .element: class="fragment" -->
* Each instruction creates a new layer in our image. <!-- .element: class="fragment" -->


### Images - instructions
* Run a command.
* Add a file or directory.
* Create an environment variable.
* What process to run when launching a container from this image.
---
### Dockerfile <!-- .element: class="fragment" -->



### Containers
* Container is built from an image.
* A container consists of an operating system, user-added files, and meta-data.<!-- .element: class="fragment" -->
* That image tells Docker what the container holds, what process to run when the container is launched, and a variety of other configuration data.<!-- .element: class="fragment" -->
* The Docker image is read-only.<!-- .element: class="fragment" -->
* When Docker runs a container from an image, it adds a read-write layer on top of the image in which your application can then run.<!-- .element: class="fragment" -->



### CLI Examples
```
$ docker --help
``` 

``` 
$ docker version
```
<!-- .element: class="fragment" -->


run a container 
``` 
$ docker run ubuntu:18.04 /bin/echo "Hello World"
```
what happened?  
* Download the image `ubuntu:18.04`
* Create a new container
* Execute the `echo` command within the new container


run as service 
``` 
$ docker run -d ubuntu:18.04 /bin/sh -c "while true;\  
 do echo Hello World; sleep 1; done"
```
check running containers <!-- .element: style="text-align:left;" class="fragment" data-fragment-index="1"-->
```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
297d3c45afc4        ubuntu:18.04        "/bin/sh -c 'while "   5 hours ago         Up 5 hours                              adoring_wilson
```
<!-- .element: class="fragment" data-fragment-index="1"-->
see logs <!-- .element: style="text-align:left;" class="fragment" data-fragment-index="2"-->
```bash
$ docker logs 297d3c45afc4
```
<!-- .element: class="fragment" data-fragment-index="2"-->
stop running containers <!-- .element: style="text-align:left;" class="fragment" data-fragment-index="3"-->
```bash
$ docker stop 297d3c45afc4
```
<!-- .element: class="fragment" data-fragment-index="3"-->


### More options
port forwarding <!-- .element: style="text-align:left;" -->
``` bash
-p hostPort:containerPort
```
volume mounts <!-- .element: style="text-align:left;" class="fragment" data-fragment-index="1"-->
```bash
-v hostFolder:containerFolder
```
<!-- .element: class="fragment" data-fragment-index="1"-->
e.g. Mount local directory /html as directory /usr/share/nginx/html in the container<!-- .element: style="text-align:left;" class="fragment" data-fragment-index="1"-->
```bash
-v /html:/usr/share/nginx/html
```
<!-- .element: class="fragment" data-fragment-index="1"-->



### Demo time
![pr](./content/images/coding.gif) 


### Exercise
----------
1. Start a webserver
2. Overwrite the content of the index.html
3. Watch the webserver logs
4. compare the output of `ps -ef` from your container with the host
> https://www.katacoda.com/courses/docker/create-nginx-static-web-server


### Dockerfile
Dockerfiles are the blueprint instructions for building images
* are usually called `Dockerfile`
* Contain a list of instructions, which are executed from top to bottom


### Dockerfile
* The build has the current directory as context
* All paths are relative to the `Dockerfile`
* Each command in the `Dockerfile` creates a new (temporary container)
* Each command in the `Dockerfile` is another layer in the overlay filesystem
* Every creation step is cached, so repeated builds are fast


#### FROM
The `FROM` instruction sets the Base Image:

    FROM <image>:<tag>

Example:

    FROM nginx:15:04


### COPY
`COPY` can be used to copy files or directories to the image.

    COPY <src>... <dest>

- Source can contain wildcards
- If dest does not exist it will be created

Example:<!-- .element: style="text-align:left;" -->

    COPY service.config /etc/service/
    COPY service.config /etc/service/myconfig.cfg
    COPY *.config /etc/service/
    COPY cfg/ /etc/service/


### CMD
With `CMD` you can specify the default command to execute on container startup.

    CMD ["executable","param1","param2"]

Example:<!-- .element: style="text-align:left;" -->

    CMD ["nginx", "-g", "daemon off;"]

ADD works almost the same as copy <!-- .element: style="text-align:left;" -->


### RUN
The RUN command allows to execute arbitrary commands in the container, which modify the
file system and produce a new layered container.<!-- .element: style="text-align:left;" -->

    RUN <command>

It is common to tie related commands together into one RUN command, using shell magic.<!-- .element: style="text-align:left;" -->

Example:<!-- .element: style="text-align:left;" -->

    RUN apt-get update && \
        apt-get install -y ca-certificates nginx=${NGINX_VERSION} && \
        rm -rf /var/lib/apt/lists/*


### ENV
`ENV` sets environment variables which are present during container build and remain existent in the image.

    ENV <key> <value>
    ENV <key>=<value> ...

On container startup they can be overwritten with the `-e` or `--env` option:

    docker run -e key=value my_image


### Building images
```shell
$ docker build -t yehiyam/my-image:v1 .
```
To specify the Dockerfile<!-- .element: style="text-align:left;" -->

```shell
$ docker build -t yehiyam/my-image:v1 -f ./folder/Dockerfile .
```


### Registry
* On of the strengths of Docker is the ability to easily share and distribute images
* A Docker registry holds image in a central location to allow easy sharing
* The default registry is Docker Hub


### Docker Hub
* https://hub.docker.com/
* Free for public repositories (public read only)
* Need to login to push
```
$ docker login -u username -p password
```
* To push to the registry
```
$ docker push yehiyam/my-image:v1
```


### Private Registries
* Docker provides a Docker image we can use to host our own registry
* Specify the registry before the image name
```shell
$ docker push my-registry:5000/yehiyam/my-image:v1
```
Works for tagging, pulling and pushing


### Insecure Registries
* By default Docker access the registry with HTTPS
* When setting up a private registry in the organization, it is usually HTTP
* To access the registry we need to tell the Docker engine that it is OK
```shell
/etc/docker/daemon.json
```
```json
{
    "insecure-registries": [
        "my-registry:5000",
        "your-registry:9999"
        ]
}
```


### Exercise
----------
1. TBD

