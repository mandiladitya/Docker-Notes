# Overview

[Docker](https://www.docker.com/) is a software to run containers, that is you can run one or several other operating systems on your computer.  It runs natively (except on Mac, where it uses a VM), making it faster and smaller than a VM.  For development, it allows you to reproduce an environment, which has benefits in testing and for deployment.

Here is a good description of containers from [here](https://www.backblaze.com/blog/vm-vs-containers/):

> Containers sit on top of a physical server and its host OS — typically Linux or Windows. Each container shares the host OS kernel and, usually, the binaries and libraries, too. Shared components are read-only. Sharing OS resources such as libraries significantly reduces the need to reproduce the operating system code, and means that a server can run multiple workloads with a single operating system installation. Containers are thus exceptionally light — they are only megabytes in size and take just seconds to start. Compared to containers, VMs take minutes to run and are an order of magnitude larger than an equivalent container.

![](https://i1.wp.com/www.docker.com/blog/wp-content/uploads/Blog.-Are-containers-..VM-Image-1-1024x435.png?ssl=1)

Graphic showing the differences between running containers and virtual machines.

![](https://github.com/audrow/docker-notes/blob/master/imgs/docker_vs_vms_comparison.png?raw=true)

Summary of differences

# Setup

Instructions to install Docker:

[About Docker Engine - Community](https://docs.docker.com/install/)

To make it so that you don't have to run `sudo` everytime you run a Docker command, give it group access (see [here](https://askubuntu.com/a/477555))

    sudo groupadd docker
    sudo gpasswd -a $USER docker
    newgrp docker # or logout to apply changes

# Docker basics

You can run Ubuntu 19.04 with the following Docker command:

    docker container run -it --rm ubuntu:19.04 bash

The `-it` makes it so that you are given an interactive terminal (kind of like SSH'ing into the container), and  `--rm` removes the container after you leave the container. `ubuntu:19.04` is the docker image that you will build from [Docker Hub](https://hub.docker.com/), a collection of Docker images.  In this `ubuntu` is the Docker repository and `18.04` is the tag.  You could also do `16.04` or any other version of Ubuntu that there is a tag for (see [here](https://hub.docker.com/_/ubuntu?tab=tags)).

[Here](https://github.com/wsargent/docker-cheat-sheet) is an excellent and more complete reference.

# Dockerfile

A Dockerfile (literally named, `Dockerfile`, with no extension, by default) is something that specifies how to build a Docker image.  Each line in a Dockerfile has a command like `RUN`, `CMD`, `COPY` and creates another "layer."  A layer can be thought of as an intermediate Docker image (like a class, not an instance), and you build your Docker image, which makes Docker containers (the instance), by adding one or more layers on existing Docker images (like Ubuntu).

Docker uses a caching system when building containers, making it so that each line in a Dockerfile is saved.  This means that running a Dockerfile the second time is almost instant.  It also means that when writing your Dockerfile, try not to change lines that are at the top of the file because it will have to rebuild everything below it.

Some notes:

- Don't use `sudo` because running a Dockerfile creates an image with super user permissions anyway.  If you want to run commands as a user in the Dockerfile and to use `sudo`, you will have to install it with `apt install` first.
- No commands run from a Dockerfile must require user input, if they do, the Dockerfile will fail to build.  Instead, automatically accept, for example `apt install -y curl`.
- It is good practice to make several installs at once (and even several statements), you can use `\` to continue on a new line and `&&` to run another command after the last one succeeds.  For example:

        RUN apt update && apt upgrade -y \
        	&& apt install -y \
        		curl \
        		wget \
        		tmux \
        	&& rm -rf /var/lib/apt/lists/* # best practice to remove apt install stuff

Here is a minimal example:

    # Saved as `Dockerfile`
    FROM ubuntu:19.04

    RUN apt update && apt upgrade -y \
    	&& apt install -y \
    		curl \
    		wget \
    		tmux \
    	&& rm -rf /var/lib/apt/lists/* # best practice to remove apt install stuff

    CMD ["bash"]

From command line

    cd <directory/with/the/dockerfile>

    # Build the docker image from the Dockerfile
    docker image build -t example .

    # See that the image called example:latest has been created
    docker image list

    # Run the created image
    container run -it example
    # Note that we do not need to call bash to run bash as it was set to run with CMD in the Dockerfile
    # Also, latest is the default tag if you don't specify
