<!-- from Docker Up And Running (Second Edition) by Sean Kane / Karl Matthias -->


# Chapter 1: Introduction

Prior to Docker:

  * You'd have to know `rpm`, `mock`, `dpkg`, `pbuilder`, etc. to create software packages for a supported platform. Dockerfiles introduce a standard interface/methodology for that problem.
  * It was arduous to bundle applications with dependencies - layered images solve that problem.
  * The entire Docker image is an artifact that does not have to be recompiled throughout deployments.
  * Virtual machine hypervisors request a % of the host machine's compute/memory resources, whereas Docker requests resources directly from the kernel, which is more efficient.

Containers and their hosts share the same kernel. Docker *lessens* but does not obviate the need for configuration management. Docker is deployment-agnostic.


# Chapter 2: The Docker Landscape

Developers used to request that operations teams perform deployments and environment bootstrapping.

Docker, Inc. sponsored the Open Container Initiative because build platforms really needed an immutable specification to work with before they committed time/energy/effort to adopting Docker.

The `docker` CLI builds images, pulls images, starts containers, retrieves remote Docker logs, monitors stats, etc.

Each stacked filesystem layer in a Docker container has a unique hash.


# Chapter 3: Installing Docker

This chapter seems to have been written for the sole purpose of increasing the page count of the book.


# Chapter 4: Working with Docker Images

In a Dockerfile:

  * `FROM` specifies a base image.
  * `LABEL` specifies arbitrary metadata.
  * `USER` specifies the user that the container processes runs as. The default is `root`.
  * `ENV` sets environment variables.
  * `RUN` specifies shell commands to execute *during the image build procss*.
  * `ADD` copies files from a local or remote filesystem into an image.
  * `WORKDIR` changes the working directory for the purposes of `RUN` commands.
  * `CMD` specifies shell commands to run *on container start*.

`.dockerignore` files specify which files/directories should not upload to the Docker host during a build.

During a `docker build`, every instruction is one *layer* of the overall Docker image layering.  You can actually `docker run` into a lower container image (not the resulting one that you deploy!).

<!-- todo: page 68 -->