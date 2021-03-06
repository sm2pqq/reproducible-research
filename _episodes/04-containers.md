---
layout: episode
title: "Containers"
teaching: 10
exercises: 10
questions:
   - "How to capture the software environment of a computational experiment?"
   - "How can we communicate different versions of software dependencies?" 
objectives:
  - "Get a basic idea of using containers to capture research software environments"
keypoints:
  - Use containers to share research software environments

---

# Reproducible environments

- Software may have lots of dependencies which may be difficult to recreate.
- Results should be possible to reproduce regardless of platform and with minimal effort.
- Many research codes can be problematic to install and configure without experts.
- Can we bundle all the necessary dependencies together, making it easier to run the software?

# Containers

- Containers can be built to bundle all the necessary ingredients (data, code, environment).
- A great solution to the problem of ["dependency hell"](https://en.wikipedia.org/wiki/Dependency_hell).
- Allows for seamlessly moving workflows across different platforms.
- A container provides operating-system-level virtualization, i.e. it shares the host system’s kernel with other containers.
- Popular container implementations are **[Docker](https://www.docker.com/)** and **[Singularity](http://singularity.lbl.gov/)**.
- "[the term] is borrowed from Shipping Containers, which define a standard to ship goods globally. Docker defines a standard to ship software." ([from the Docker documentation](https://docs.docker.com/glossary/)).

## Docker

- Docker provides containerization in software level.
- Available for most common operating systems.
- Provides an easy and fast way to bundle all the necessary libraries and data together.
- DockerHub is a platform to share Docker images (stored in repositories - similar to Git repository).
- Public Docker images available on [DockerHub](https://hub.docker.com/) but a word of warning: <span style="color: red">not all images can be trusted! There have been examples of contaminated images so investigate before using images blindly</span>.

---

## Singularity 

- [Singularity](http://singularity.lbl.gov/) is aimed at scientific community and to run scientific workflows on HPC resources.
- Docker is compatible with Singularity:
  - Docker images can be converted into Singularity images.

---

## Docker fundamentals

- Docker is a client-server application. The Docker client talks to the Docker server
or daemon, which, in turn, does all the work.

<img src="/reproducible-research/img/docker_architecture.svg" style="height: 400px;"/>

- Docker client
   - End user uses Docker client to communicate with Docker daemon .
- Docker daemon
   - Executes the commands sent to the Docker client. Manages containers, images, builds, etc.
- Docker registry
   - Stores Docker images. DockerHub and Docker Cloud are public registries that anyone can use, and Docker is configured to look for images on DockerHub by default.
   - You can even run your own private registry.


## What are Docker images and containers

- A Docker *image* is executable package of a piece of software that includes everything needed to run it: code, runtime, system tools, system libraries, settings.
- When you start the image, you have a running *container* of this image.

Getting help:
```shell
$ docker help <command>
```

Listing Docker images:
```shell
$ docker images
```
Searching Docker images from DockerHub:
```shell
$ docker search ubuntu
```
Pulling from DockerHub (pulls the latest one by default if no tag is mentioned):
```shell
$ docker pull ubuntu
```
Starting container from the pulled image:
```shell
$ docker run -i -t ubuntu
  
-i flag keeps STDIN open from the container
-t flag provides an interactive shell to the container
```
   
Check running containers:
```shell
$ docker ps
```
Check all containers (also those not running):
```shell
$ docker ps -a
```
Stop the container:
```shell
$ docker stop container_id or name
```
Start (in interactive mode) a stopped container:
```shell
$ docker start -i container_id or name
```
Remove a container:
```shell
$ docker rm <container name>
```
Remove an image:
```shell
$ docker rmi <image name>
```

## Building Docker images

- An image is built based on a Dockerfile.
- A Dockerfile contains a series of instructions paired with arguments.

```vim
#version 0.1
FROM ubuntu:16.04 (good to mention the image version being used)
LABEL maintainer="sriharsha.vathsavayi@csc.fi"
RUN apt-get update
...
```
Instructions in the Dockerfile (for full reference, please visit [Dockerfile](https://docs.docker.com/engine/reference/builder/)).

```vim
FROM -  sets the base image for subsequent instructions
RUN - execute any commands in a new layer on top of the current image and commit the results
COPY - copies local files from build context into our image
WORKDIR - provides a way to set the working directory for the container when a container is launched from the image
CMD -  specifies the command to run when a container is launched
LABEL - adds metadata to an image and is a key-value pair
..
..
```

#### Mnemonics

- The Dockerfile is like a cooking recipe, building an image from the Dockerfile 
  is like doing the actual cooking.
- Running the Docker image to create a container doesn't have a good cooking analogy, but:
  - a Docker image is like a *class* in OOP, a Docker container is like an instance of the class.

  
## Type-along exercise: Containerizing our workflow

> This exercise is based on the [same example project](https://github.com/coderefinery/word-count) as in the previous episodes

Let's create a Dockerfile for our example project.  
It is available in the project repository if you want to experiment with 
it later.

```vim
#version 0.1
FROM ubuntu:16.04

#maintainer information
LABEL maintainer="kthw@kth.se"

# update the apt package manager
RUN apt-get update
RUN apt-get install -y software-properties-common
RUN add-apt-repository ppa:jonathonf/python-3.6
RUN apt-get update

# install make
RUN apt-get install -y build-essential

# install nano
RUN apt-get install -y nano

# install python
RUN apt-get install -y python3.6 python3.6-dev python3-pip python3.6-venv
RUN yes | pip3 install numpy
RUN yes | pip3 install matplotlib
RUN yes | pip3 install snakemake

# copy project to container
COPY ./ /opt/word_count/

# set work directory in container
WORKDIR /opt/word_count

# default command to execute when container starts
CMD /bin/bash
```

We can build the image by running docker build in the word_count directory containing Dockerfile:

```shell
$ docker build -t word_count:0.1 .
``` 
This will take a few minutes...

Check if the image is created:
```shell
$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
word_count          0.1                 3103c7bde05b        4 minutes ago       744MB
ubuntu              16.04               7aa3602ab41e        3 weeks ago         115MB
``` 

- We now have two images, the *base image* `ubuntu:16.04` is the parent of our `word_count:0.1` image.


## Starting containers from images

We can run a container using `docker run` command:

```shell
$ docker run -i -t --name wordcount word_count:0.1
``` 
Note: Use -d to start a container in the background in a detached mode (to create long-running containers).

We can now see the running container (in another terminal):
```shell
$ docker ps
```

---

## Managing data

- Docker containers should be disposable - the data must be saved elsewhere.
- A Docker volume allows data to persist, even when a container is deleted. Volumes are also a convenient way to share data between the host and the container.
- For details on volumes, [refer to the documentation](https://docs.docker.com/engine/admin/volumes/) page.

**Sharing a host directory with container**
 
  ```shell
$ docker run -it --name my-directory-test -v <path-on-hostmachine>:/opt/data <image_name>
  ```

**Anyone with this image can reproduce the results we have generated**
  ```shell
$ docker run -v <path-on-hostmachine/results_directory>:/opt/word_count/results word_count:0.1 snakemake -s Snakefile_all
  ```
The `results_directory` folder will have the results of our word count example project.

We can also specify snakemake (or  any other command) as the default command to run when our container starts, by giving it as parameter for CMD in Dockerfile. 

## Sharing a Docker image
- DockerHub - A platform to share Docker images.
- Login to DockerHub:

 ```shell
$ docker login
  ```
- Push to DockerHub. The image name has to be in **youruser/yourimage** format 
(thus instead of the name `word_count`, 
we should have used `<dockerhub-username>/word_count` above):

 ```shell
$ docker push image_name
  ```
- For proprietary/sensitive images private Docker registries can be used

---

### Where you might want to go from here

- New platforms for doing reproducible research are emerging, see for example [**reana**](http://www.reana.io/).
- Good talks on open reproducible research can be found [here](http://inundata.org/talks/index.html).
