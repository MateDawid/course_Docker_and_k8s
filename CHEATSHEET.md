# Images
## Pull image
```sh
docker image pull <image-name>
```

## Build image
```sh
docker image build -t <image-tag> -f <path-to-dockerfile> <path-to-exec-dir>
# Examples
docker image build -t my-nginx -f nginx/Dockerfile nginx/
docker image build -t my-image -f docker/Dockerfile .
```


## Containers
## Run container
```sh
docker container run --name <container-name>  <image-name>
#Examples:
# -P -> Picks random available host port
docker container run --name my-python-service -P -d my-python
# -d -> Runs container in detached mode
docker container run --name my-python-service -P -d my-python
# -e -> Runs container with additional environment variables
docker container run -e REDIS_HOST=192.160.0.2 -d -P -e LOG_LEVEL=DEBUG --name my-python-service my-python
# -p -> <host-port:container-port>
docker container run -d -p 5002:5002 --name my-python-service my-python
```

## List containers
```sh
docker container ls
docker container ls -a
docker container ls -a -f status=exited
docker container ls -a -f label=type=workshop
```

## Check logs
```sh
docker container logs <container-name|container-id>
```

## Remove container
```sh
docker container rm <container-name>
```

## Remove all containers
```sh
docker rm -f $(docker ps -q) 
docker rm -f $(docker ps -q -a) 
```

## Exec command in container
```sh
# -it stands for interactive terminal
docker containet exec -it <container-name> <command-to-exec>
# Example
docker container exec -it my-nginx /bin/sh
```

## Stop container
```sh
docker container stop <container-name>
```

## Container port
```sh
docker container port my-python-service
```

## Diff
Displays diff between image and container files.
```sh
docker container diff my-nginx
```

## History
```sh
docker image history my-nginx
```

## Inspect
```sh
docker container inspect my-redis
```
# Volumes
## List volumes
```sh
docker volume ls
```

## Create volume
```sh
docker volume create my-volume
```

## Run container with volume
```sh
# -v <host-dir:container-dir>
docker container run --rm -v /tmp:/data my-volume /data
# -v <volume-name:container-dir>
docker run -v my-volume:/data -d --name container2 my-nginx
# -v <volume-name:container-dir>:ro -> Read only access to volume
```

# Networks
## List networks
```sh
docker network ls
```
## Create network
```sh
docker network create python-redis-network
```

## Run container in network
```sh
docker container run --network python-redis-network -d --name my-redis redis
```

# docker-compose

