# Containers

## History

In the 1960s-70s, CP/CMS was a mainframe operating system that allowed for "time sharing", which was a method that allowed multiple users to access the mainframe's CPU/storage concurrently. Each user got their own virtual operating system within the machine.

In 1979, `chroot` was developed for Unix, which allowed you to change the root directory for a running process and its children - so the process is sandboxed insofar as it doesn't know the rest of the file system exists. However, `chroot` did not sandbox hostnames and IP addresses, it can't run as `su`, and the directories must be owned by `root:root`.

In 2000, FreeBSD introduced Jails, which allowed for a Unix system to be partitioned with separate users, processes, file systems, and networking stacks - but it was difficult to use.

Sun Solaris and HP-UX were early forerunners of multiple isolated virtual environments. Later, VMWare developed hypervisors like ESXi, which allow multiple virtual machines to run on a single physical machine. 

In 2002, the Linux kernel added support for `namespaces`, which partition kernel resources across:

  1. `user`, which provide privilege isolation and user ID segregation across sets of processes
  2. `pid`, which allowed processes in a namespace to have their own unique process IDs
  3. `network`, which allowed processes in a namepace to have their own networking stacks, and which allowed processes to communicate with one another
  4. `mount`, which allowed processes to mount filesystems
  5. `uts`, or Unix Time Sharing, which allowed for processes to have isolated hostnames/domains
  6. `ipc`, which set up message queues for processes
  
In 2006, Google developed `cgroups`, also known as *process containers*. These isolate resource limits (CPU / memory / network allocation / I/O), priority allocations, monitoring/reporting of resource limits, and process control (start/stop/frozen/restarted).

In 2010, Docker was founded, leveraging `namespaces` and `cgroups`, to define container runtimes, which share the same kernel as the host operating system. Containers receive their own users / hostnames / IPs / mounts / resource allocations.

## Overview

Container images are portable, self-contained bundles of software along with dependenies. Container images are OCI-compliant - meaning that they can run on Docker, K8S, AWS ECS, etc. A container is an *instance* of a container image - an Nginx container image can spawn multiple redundant Nginx containers. Images are stacks of layers overlaid on one another. 

Image tags distinguish versions of container images. The `latest` tag is the default.

## Docker

Docker commands generally follow `docker [noun] [verb]` syntax.

`docker run -it` means start a *interactive*, *teletype interface* session with a container (i.e. using a terminal).

  * `--rm` will remove a container once it exits.
  * `-d` runs in the background (i.e. *detached*).
  * `-P` publishes all ports, which is like the `EXPOSE` directive in a Dockerfile, which exposes all ports (using a random port).
  * `-p 12345:80` will publish a container's ports to the host. The first parameter is the *host port*, and the second parameter is the *container port* to which it is mapped. `12345` in this case is the external port used to access some application running on port `80` within the running container.
  * `-v` provides a volume mount so that a running container can access the file system of the host machine running the container.

`docker images` shows local images. Image digests differ from image IDs - a *digest* is a checksum from a container registry and an image ID is a checksum of the local container image.

`docker manifest` shows the mainfest from some registry, along with layers.

`docker save` saves an image to a local file for backup/import.

`docker version` shows the local Docker version. Docker Desktop uses `containerd` and `runc`, which have their own versions.

`docker pull foo/bar` downloads a container from DockerHub, or another registry if you specify one.

`docker ps` shows you running containers. `docker ps -a` will show containers that have exited.

`docker rm` accepts a container ID or name, and it removes a container.

`docker stop` will stop a running container.

`docker exec` will execute a command from within a running container. `docker exec -it [container ID] bash` will start an interactive bash session within a running container.

`docker build . -t foo/bar` builds the current directory to an image.

`docker login` logs into DockerHub.

`docker buildx` is used to build a Docker container to different processor architectures.

`docker rmi` removes a local image.

`docker system prune` removes stopped containers, networks, dangling images, caches, etc.

## Dockerfiles

Dockerfiles are used to build images.

One approach to building a Dockerfile is to start with a blank base image, start an interactive session with it, do whatever you need to do, and use `history` to get a list of the commands used to bootstrap the container, and copy those commands into a Dockerfile. 

Docker caches layers used in builds - the fewer layers, the better for build times. Multi-stage builds reduce the size of the final container image. 

  * You start with a `FROM` directive which specifies a base image (like `alpine`).
  * `MAINTAINER` is used to specify who is maintaining a container. `LABEL` is an alternative. 
  * `RUN` executes shell commands that run when the container is built.
  * `CMD` executes shell commands that run when the container *starts*.
  * `WORKDIR` creates a directory if it doesn't exist, and provides a filesystem context for `RUN` commands.
  * `COPY` can be used to borrow things from other containers, thus making a child (or inherited) container a bit smaller.
  * `USER` specifies a non-root user for the container to run in. Note that the user has to actually exist.
  * `ENTRYPOINT` specifies that any CLI arguments passed to `docker run [image]` will then be passed to some executable running in the Docker container. For example: if `CMD` needs arguments at `docker run`-time, then you should replace `CMD` with `ENTRYPOINT`.

## Orchestration

Container orchestrators provision and deploy containers, ensure availability / self-healing, scheduling compute resources, expose container services, provide authorization and security, provide storage for shared workloads (and persistent workloads), auto-scale, and provide extended functionality (such as CRDs, or *custom resoure definitions*).

Some container orchestration tools include OpenShift, Docker Swarm, HashiCorp Nomad, and Kubernetes.