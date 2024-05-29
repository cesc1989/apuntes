# Dive into Docker Course
Lecciones del curso de Nick Janetakis.

## General
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
- set `COMPOSE_PROJECT_NAME=` env var so that docker-compose doesn't prepend containers, images, networks, volumes, etc, with folder name but with the value in the variable
- [docker-compose.yml file reference](https://docs.docker.com/compose/compose-file/)
- Ejecutar comandos `rails c` y `psql` en contenedores


## Virtual Machines are like houses. Docker Containers are like an apartment building
- [Docker Website What is a Container section](https://www.docker.com/what-container)


# Docker Image vs Docker Container

In terms of POO, an image is a class and a container is an instance of that class:

- **docker image**:
    - everything you need to run an application
    - you can download it, build it and run it
    - one image can launch many containers
- **docker container**:
    - are immutable: change made during runtime are lost when stopped
# Difference between cloud.docker.io and hub.docker.io
- [Quora](https://www.quora.com/unanswered/Whats-the-difference-between-Docker-Cloud-and-Docker-Hub)
- [cloud.docker.io](http://cloud.docker.io) is a Cloud Provider for managing **docker containers**
- [hub.docker.io](http://hub.docker.io) is a repository for sharing **docker images**


## Error response from daemon
    Error response from daemon: Get https://registry-1.docker.io/v2/: unauthorized: incorrect username or password

Respuesta en [Stack Overflow](https://stackoverflow.com/questions/40872558/just-created-docker-hub-account-credentials-do-not-work-for-docker-login).

> docker login subcommand only allows **username** and not **email**


# Docker Registry is almost GitHub but for docker images

A docker repository contains one or more docker images. Docker images are tagged for referencing defined versions.

# Docker Image

A docker image is a stack of one or more layers. Each layer can be seen as a self-contained file.

A docker repository contains one or more docker images. Docker images are tagged for referencing defined versions.

## docker image commands annotations

> Remember: you can see help of every command with the `--help` option.
> 
> There are commands and management commands. Normally, you'd use **management** commands alongside **commands**. Some are shortcuts.

When you define a Dockerfile you can create an image based on its content using:
```bash
docker image build -t [NAME:TAG] PATH
docker image build -t bucket_api:1.0 .
```

Build with env vars:
```bash
docker image build -t [NAME:TAG] --build-arg ENV=VALUE -f Dockerfile PATH
docker image build -t pruebados --build-arg SIDEKIQ_LICENSE_KEY=xxxxx:bbbbb -f Dockerfile .
```

That image can be uploaded to the Docker registry. But remember to tag it with the namespace format for the Docker Hub:

    DOCKER-HUB-USERNAME/IMAGE-NAME[:TAG]
    cesc1989/bucket_api:latest

Tagging commands:

    docker image tag \[IMAGE-NAME\] [TAG]
    docker image tag base cesc1989/bucket_api:latest

You can see a list of all images with `docker images` command; a list of all running containers with `docker container ls` or a list of all containers, running or not with `docker container ls -a`.


## `docker container` commands annotations

> Taken from Dive into Docker course

    docker container run [OPTIONS] IMAGE
    docker container run -it -p 5000:5000 -e FLASK_APP=app.py web1

`-e` is for environment vars; `-it` is for interactive console; `-p` indicates host port and docker container(guest) port in the form of `HOST-PORT:GUEST-PORT`

Delete the container after it's stopped

    docker container run -it -p 5000:5000 -e FLASK_APP=app.py \
     --rm --name web1 web1

`--rm` flag indicates to delete it after being stopped; `--name` assigns a name for better identifying the container

Run it on detached mode

    docker container run -it -p 5000:5000 -e FLASK_APP=app.py \
     --rm --name web1 -d web1

`-d` flag indicates the container to run on detach mode

Run it so that it restarts itself after a system failure

    docker container run -it -p 5000 -e FLASK_APP=app.py \
     --rm --name web1_2 -d --restart on-failure web1

Notice that when only passing a value to `-p` docker assigns a random port to HOST.

You can run many containers based of the same docker image. Do it by changing their name so they're unique from each other

    docker container run -it -p 5000 -e FLASK_APP=app.py \
     --rm --name web1_2 -d --restart on-failure web1
    
    docker container run -it -p 5000 -e FLASK_APP=app.py \
     --name web1_3 -d --restart on-failure web1


## `docker container` with volumes annotations

Docker containers have ephemeral storage, meaning every file created while the container is running will be lost after it's stopped. In order to avoid data loss, volumes must be used.

Another case scenario for volumes is live reloading of code. Without a volume, there's no synchronization between the docker host and the docker container.

Run a docker container with a volume

    docker container run -it -p 5000:5000 -e FLASK_APP=app.py \
    -e FLASK_DEBUG=1 --rm --name web1 -v $PWD:/app web1

The `-e` flag defines `FLASK_DEBUG=1` environment variable used by the framework to show a debug log of every request and trace; the `-v` flag indicates a volume to be created at in the form of:

    HOST:GUEST
    $PWD:/app

Create a file in the volume

    docker container exec -it --user "$(id -u):$(id -g)" \
    web1 touch hi.txt


> Note: there's a command called: `id`
> Note: the `exec` needs a running container


## `docker network` commands annotations

Networking commands are under the `network` management command. By default, all containers, unless specified, are tied to the `bridge` network.

With `docker network inspect bridge` you can see the JSON object representing the network data. That object contains a `containers` key which references all containers inside that network.

One can see a container IP address (required for networking different containers) with the `ifconfig` command:

    docker exec redis ifconfig # 172.17.0.2 look for the IP in the output of eth0
    docker exec web2 ifconfig # 172.17.0.3

And you can even `ping` a container from another one by `ping`ing its IP address:

    docker exec web2 ping 172.17.0.3


> Note: you can see a container hosts file: `docker exec redis cat /etc/hosts` and notice it contains a reference of the container's IP address and the container ID (the hash)

Now, if you want to control more aspects of the network or create containers in a different one, you can create a `docker network` and then tied containers to it

    docker network create --driver bridge firstnetwork

Now, you can specified a network where to launch containers

    docker container run --rm -itd -p 6379:6379 --name redis \
    --net firstnetwork redis:3.2-alpine
    
    docker container run --rm -itd -p 5000:5000 -e FLASK_APP=app.py \
    -e FLASK_DEBUG=1 --name web2 --net firstnetwork -v $PWD:/app web2

Notice how you can issue technology-specific commands with the `exec` command:

    docker exec -it redis redis-cli
    docker exec -it alpine sh


## `docker volume` for a data storage commands

For a normal docker container a volume can be assigned by setting it using the `-v` flag to the current working directory as in:

    docker container run --rm -itd -p 5000:5000 -e FLASK_APP=app.py \
    -e FLASK_DEBUG=1 --name web2 --net firstnetwork -v $PWD:/app web2

but for a data storage this is somewhat different. You'd still need to create a volume but the linking process is quite different. Using named volumes:

    docker volume create web2_redis
    
    docker container run --rm -itd -p 6379:6379 --name redis \
    --net firstnetwork -v web2_redis:/data redis:3.2-alpine

By inspecting the created volume you can see where in the docker host the data is going to be saved in the `Mountpoint` key.

The location in the docker container is defined by the image. Always check data storage image README to know where or how to set the volume for the container.


> Note: persistent storage databases such as PostgreSQL or MySQL do save data but Redis is an in-memory data store and that's why it looses data on container restart


    docker exec redis redis-cli SAVE

When dumping data from a data storage, it is saved in the docker host volume mountpoint.

## `CMD` & `ENTRYPOINT`

`CMD` instructions are always send as arguments to `ENTRYPOINT`. `/bin/sh -c` is the default `ENTRYPOINT` for every container.

`ENTRYPOINT` executes custom scripts and do not add image layers.


## Tips and Tricks

Stop all containers at once

    docker container stop $(docker container ls -a -q)

Clean images, containers, so on

    docker system prune

Remove all stopped containers - [Coderwall](https://coderwall.com/p/ewk0mq/stop-remove-all-docker-containers)

    docker stop $(docker ps -a -q)
    docker rm $(docker ps -a -q)

Remove all images without tag and/or repository - [Quora](https://www.quora.com/How-do-I-automatically-remove-all-docker-images-without-a-tag-or-a-repository)

    docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
    
    docker rmi $(docker images | grep "<none>" | awk "{print $3}")


## Ejecutar comandos que normalmente correría fuera del contenedor

Ingresar a la consola de postgres

    docker container exec -it [CONTAINER-NAME] psql -U bucket
    docker container exec -it bucket_postgres_1 psql -U bucket

Ingresar a la consola de rails

    docker container exec -it [CONTAINER-NAME] rails c
    docker container exec -it bucket_api_1 rails c

