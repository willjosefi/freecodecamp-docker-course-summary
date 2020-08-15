# What's this?
A summary of **freeCodeCamp - Docker Tutorial for Beginners - A Full DevOps Course on How to Run Applications in Containers**, which you can watch here: [https://youtu.be/fqMOX6JJhGo](https://youtu.be/fqMOX6JJhGo)

The author has a very good approach, is a quick process explainer and has a course website [https://kodekloud.com](https://kodekloud.com/)

----

# Introduction

## Why you need Docker?
- No more long setup ENV or VM time
- Different ENVs without a headache
- No more issues about compatibility between OS versions, libraries, dependencies
- Compared to a VM: quicker, less resources expensive

## What are containers?
- Completely isolated ENV with it your own network interfaces and mounts (volumes)
- Shares same kernel

> The methodology exists for a long time, docker came and turns it easier.

## How's it done?
You have Docker Registry Hub ([https://hub.docker.com](https://hub.docker.com)) which has a large database of images ready to use like:
```
docker run ansible
docker run mongodb
docker run nodejs
```

## Docker in DevOps
Developers can inject the infrastructure inside the codebase, linking the development and the ops. It's not required anymore write a manual on how to install/boot the application.

# The main: docker run <image>
It runs any image and creates a container. First try do find locally and if not exists, download it.

When you run some image as this, you will be in the `foreground` or `attached` mode and you will see the console output. You can't interact with terminal besides see the log output.

You will be on this console forever until you quit the container (`CTRL+C`) or the application which you run exits.

Running the container in background (`dettached mode`) you need to append the `-d` flag: `docker run -d image`. Container will run in background and the terminal prompt from your computer will be back to you.

If you ran the some container in the dettached mode and want to go to attached mode you can do this `docker attach container_name`.

## Tags
Sometimes you want to run some older versions of the image or a different image based on a tag. You should use a `:` which you can select between the available **tags**. Example:
- `docker run nginx:latest`
- `docker run node:14.0.1`
- `docker run redis:4.0`

## Run in interactive mode
If you want to run the container in interactive mode, where you can see the console questions and can answer them you need to run as `docker run -it some-image`

## Environment Variables
`docker run -e APP_COLOR=green image/example`

> You can see the all the environment variables in use for the container using `docker inspect`

## Notes
When you run `docker run ubuntu`. Ubuntu will boot up but it will exit immediately. If you run `docker ps` you will see that the container was exited.

Containers are made to boot up and run something. A task, service or something related. And not to boot up and wait for external commands as a VM is.

When the task finishes the container exits.

One example to run an Ubuntu image and just wait for a few seconds: `docker run ubuntu sleep 5`.

# Port mapping
Every container receives an IP by default but it's only reachable from the docker host. It's locked from external access.

To allow access to some port inside the container, you need to map some port from docker host to the container.

`docker run -p 80:5000 image/example`

This tells to docker map the port 80 from docker host to the port 5000 inside the container.

# Volume mapping
Everything wrote inside the container, which is outside a volume, will be lost when you delete the container.

You need to map a volume inside container to a place inside the docker host to persist the data.

This will auto-mount the volume inside the container: `docker run -v /some/path/docker-host:/var/lib/mysql mysql`

# Networking
When docker installs, three networks are created: Bridge, none and host

To specify a network to use with the run command you need to: `docker run ubuntu --network=bridge|none|host`

## Bridge
- This is the default for every container
- Every container can reach other containers inside this network
- To access the services inside this network you map using **Port Mapping**

## Host
- Uses the same network of the host
- Doesn't require do map ports as you can access everything inside the host

## None
- Doesn't use any network
- There's no access between the other containers and from external network

## Create your own network
```
docker network create \
  --driver bridge \
  --subnet 182.18.0.0/16 \
  custom-isolated-network
```

You can list all networks using `docker network ls`.

## Connect to services container-to-container
Docker has a built-in DNS which helps you connect between the containers using the `container_name`.

Example: `mysql.connect('container_mysql')`

# Storage

## Layered architecture
- Used in builds
- Each layer stores only the difference from the before-layer. Even if you have more than one Dockerfile, Docker always will use the same layer for them.
- These layers, used on builds, are read-only
- After container boot up, a new layer is created read-write and will contain all the data from the container
- When something changes any file inside the container which belongs to the image layer, a `COPY-ON-WRITE` is made

## Volumes
Docker has a special storage for volumes which you can create inside your docker host `(/var/lib/docker/volumes)`. This is called **Volume Mounting**

> It's different from **Bind Mounting**, where you can points to any place on docker host

You can create these volumes as this: `docker volume create data_volume`

To use: `docker run -v data_volume:/var/lib/mysql mysql`

> You can create the volume on the run command as well. If the volume doesn't exist, it will be created.
>
> Example: `docker run -v data_volume_which_doesnt_exist:/var/lib/mysql mysql`

### More verbose way to mount the volumes
```
docker run \
  --mount type=bind,source=/docker-host/folder,target=/var/lib/mysql \
  mysql
```

# Creating you own image
You need to create a Dockerfile to create your own image, example:
```dockerfile
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

- After you create the Dockerfile, you need build: `docker build Dockerfile -t username/my-custom-app`
- To push to Docker Hub: `docker push username/my-custom-app`

## Understanding the Dockerfile
- Everything at the left and in uppercase is an **instruction**
- It always needs to start with a **FROM instruction** (always should inherit some image already created)
- Each line of instruction is a **layer** *(helps in the build)*
  - In case of issue, the previous layers doesn't need to be repeated
  - As in the example you are using a **COPY instruction**, only this layer will be updated when you are working with your code and building the image

## CMD vs ENTRYPOINT
Every image by default has a **CMD** or **ENTRYPOINT** instruction inside Dockerfile. This instruction tell to Docker what to do when boot up.

For example, Nginx image does: `CMD nginx`, Ubuntu does `CMD bash`.

You can change the default instruction of any image just using a combination of **FROM + CMD/ENTRYPOINT**\. Example:
```dockerfile
FROM <image>
CMD ['my-new-command', 'my-new-param1']
```
You can override the image default CMD using docker run. Example: `docker run image command-to-execute-instead-the-default`

### Accept a param from docker run
If you want to accept a param from docker run, you need to use **ENTRYPONT** instruction. Example:
```dockerfile
FROM Ubuntu
ENTRYPOINT ["sleep"]
```
Run as: `docker run ubuntu-sleeper 10`

This will boot Ubuntu and run the sleep command waiting for 10 seconds.

> Different from **CMD**, **ENTRYPOINT** accept a param from docker run and attaches to the default

You can define a default param when docker run doesn't define one. Example:
```dockerfile
FROM Ubuntu
ENTRYPOINT ['sleep']
CMD ['5']
```
***CMD** needs to be use JSON format*.

> You can override the **ENTRYPOINT** too.
>
> Example: `docker run --entrypoint <new-command> image <param>`

# Docker Compose
Used to work with more than one service/image at the same time in the same project.

## Overview
- You always have to define a version
- It creates by default the network to use inside the stack (to be used by all services)

## Docker compose example file
```yaml
version: 2
services:
  <container-name>:
    image: redis
  <another-container-name>:
    image: mysql
  <another-container-name-two>:
    image: nginx
    ports:
      - 5000:80
    depends_on:
      - db
    networks:
      - backend
networks:
  - backend:
```
To boot the entire stack you need to run `docker-compose up`

You can use the Dockerfile of some image and instruct docker-compose to use the correct file to **build**. Example:
```yaml
services:
  <app-name>:
    build: ./app # path where the Dockerfile lives
```

# Docker Engine
- It's possible to run docker-cli accessing a remote docker
- A container is in fact a process in docker host machine with a unique PID, but inside the container-process-tree it starts with PID 1
- You can limit some resources inside the container
  - `docker run --cpus=.5 ubuntu` *uses max 50% of the CPU from host*
  - `docker run --memory=100m ubuntu`

# Docker Registry
By default, any image you get is inside docker.io. Example: `docker run nginx` is in fact `docker run docker.io/nginx/nginx`

When you type only `nginx`, it's converted to `nginx/nginx` which is `<user-account>/image-name`.

You can work with private registries like AWS, Azure, GCP...:
- You need to log in `docker login private-registry.io`
- Install as `docker run private-registry.io/apps/internal-app`

# Docker Orchestration
It helps to create/reboot containers when they stop in case of failure, or the traffic gets high.

## Applications
- Docker Swarm *(less features)*
- Kubernetes *(most popular)*

# Another useful commands
- `docker ps` list running containers and some information about them
- `docker ps -a` list all containers, even exited ones
- `docker stop container_name` stops a container
- `docker rm container_name` deletes the container permanently
- `docker images` list all downloaded images
- `docker rmi nginx` remove the image. Make sure you have no containers running which being using the image.
- `docker pull nginx` only downloads the image without run the container
- `docker exec container_name <command>` run a command inside the container
- `docker inspect container_name` show s more information about the container (state, volumes, network, env vars)
- `docker logs container_name` when you are in detached mode and want to see logs from container

> In any place where you need to insert container name, you can use only a partial name, which should be different from other containers, and will be enough to docker knows which container you are trying to reach.
