# Docker/Podman Cheat Sheet

Below is a list of sample docker commands to help you through some of the labs

## Listing Docker Resources
```
docker ps - lists all the running containers
docker ps -a - lists all the running and exited containers
docker images - lists all the images
docker volume ls - lists all the volumes
docker network ls - lists all the networks
```

## Run a command in a new container:
```
docker run [IMAGE] [COMMAND]

docker run --rm [IMAGE] – removes a container after it exits.

docker run -td [IMAGE] – starts a container and keeps it running.

docker run -it [IMAGE] – starts a container, allocates a pseudo-TTY connected to the container’s stdin, and creates an interactive bash shell in the container.

docker run -it-rm [IMAGE] – creates, starts, and runs a command inside the container. Once it executes the command, the container is removed.
```

## Starting and Stopping Containers

The following commands show you how to start and stop processes in a particular container.

Start a container:
```
docker start [CONTAINER]
```

Stop a running container:
```
docker stop [CONTAINER]
```

Stop a running container and start it up again:
```
docker restart [CONTAINER]
```

Delete a container (if it is not running):
```
docker rm [CONTAINER]
```

Kill a container by sending a SIGKILL to a running container:
```
docker kill [CONTAINER]
```

## Docker Image Commands

Below you fill find all the necessary commands for working with Docker images.

Create an image from a Dockerfile:
```
docker build [URL]
docker build -t – builds an image from a Dockerfile in the current directory and tags the image
```

Pull an image from a registry:
```
docker pull [IMAGE]
```

Push an image to a registry:
```
docker push [IMAGE]
```

Create an image from a tarball:
```
docker rmi [IMAGE]
```

Load an image from a tar archive or stdin:
```
docker load [TAR_FILE/STDIN_FILE]
```

Save an image to a tar archive, streamed to STDOUT with all parent layers, tags, and versions:
```
docker save [IMAGE] > [TAR_FILE]
```

## Debugging containers

List the logs from a running container:
```
docker logs [CONTAINER]
```

List low-level information on Docker objects:
```
docker inspect [OBJECT_NAME/ID]
```

List real-time events from a container:
```
docker events [CONTAINER]
```

Show running processes in a container:
```
docker top [CONTAINER]
```