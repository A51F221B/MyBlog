---
title: Basics of Docker
date: '2023-09-08'
tags: ['Docker', 'Containers', 'Containerization','Dockerfile']
draft: false
summary: Docker is used to deploy application along with it complete dependencies.Any user can use the docker container and run the application without installing any extra frameworks or modules.

---

## Docker Notes


### Introduction

Docker is used to deploy application along with it complete dependencies.Any user can use the docker container and run the application without installing any extra frameworks or modules.

### Docker Container

A Docker Container is an isolated environment for running an application.We can have many docker containers running different versions of the same application running in isolation.Containers are very lightweight and share the operating system of the host.Or in more accurate terms the containers use the kernel of host operating system.This is one of the main difference between virtual machines and containers.On a single host we can run hundreds of containers side by side.

### Docker Architecture

Docker uses a client server architecture.The client talks to the server using a REST API.The server is also called Docker Engine sits on the background and is responsible for building and running docker containers.In some ways a container is a process running on your computer.

### Installing Docker

Docker can be installed by following the steps mentioned on the official docker website.To check the version of docker that is installed on your computer you can use the command `docker version`.

### Development Workflow

We can dockerize any application to make it run on docker.We just add a `Dockerfile` which contains all the instructions to package this application into an image.The image contains everything needed to run that application.
An image may contain
- A cut-down OS
- A runtime environment (eg. node)
- Application files
- Third-party libraries
- Environment variables

Once the image is made, we tell docker to run a container using that image.So instead of running the application directly , we can use docker and run that application in a separate and isolated environment.Once we have the image file, we can push it to docker registry.There are a lot of official images of different softwares available on dockerhub.

### How to Use Docker

Suppose we want to dockerize a `hello.py` file.This is the file.

```python
print("hello world")
```

Now typically if we want to run this application in a different system , we'd first have to make sure the python interpreter is present , and if not we will have to install that first.
So to use docker , first we have to create a file with name `Dockerfile` and write the instructions for the image.

```Dockerfile
FROM python:alpine
COPY . /app
WORKDIR /app
CMD python3 hello.py
```

After that we have to tell docker to execute this file using `docker build -t hello-docker .` where `-t` is the tag we are giving our docker image and `.` is referring to the directory we are current in.

```bash
╭─asif221b@Mac ~/Documents/docker-test
╰─$ docker image ls
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
╭─asif221b@Mac ~/Documents/docker-test
╰─$ docker build -t hello-docker .
[+] Building 24.7s (8/8) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                     0.2s
 => => transferring dockerfile: 106B                                                                                                                     0.0s
 => [internal] load .dockerignore                                                                                                                        0.0s
 => => transferring context: 2B                                                                                                                          0.0s
 => [internal] load metadata for docker.io/library/python:alpine
```

After docker successfully builds the image we can see it in the list.

```bash
╭─asif221b@Mac ~/Documents/docker-test
╰─$ docker image ls
REPOSITORY     TAG       IMAGE ID       CREATED              SIZE
hello-docker   latest    f7b6984f8937   About a minute ago   48.7MB
```

We can run this image by using the `docker run [image name]` command.

```bash
╭─asif221b@Mac ~/Documents/docker-test
╰─$ docker run hello-docker
hello world
```

To see the list of running docker containers at a given time we can use `docker ps` command.To order to start an interactive container we can use the command `docker run -it [container name]`. We can also run a container using its id.

`docker start -i [id]`

#### Components of `Dockerfile`

- FROM ->  Contains the base image
- WORKDIR -> specifying the working directory 
- COPY -> to copy files and directories
- ADD
- RUN -> to run commands
- ENV -> for setting environment variables
- EXPOSE ->  for telling docker to run on specific port
- CMD -> to specifying the command to run


