docker build -t "hello" .
docker run hello

docker ps -a

docker build -t "long" .
docker run --name specific-long-name --rm -d long

docker ps -a

docker stop specific-long-name

docker ps -a -q
docker rm \$(docker ps -a -q)

docker images -q
docker rmi \$(docker images -q)

docker inspect <id>
docker run --rm -p <HOST_PORT>:<CONTAINER_PORT> -d nginx
docker run --rm --name nginx-zx-1 -p 8080:80 -d nginx

docker exec -it nginx-zx-1 /bin/bash

docker run --name redis-1 --rm -v </path/to/host/folder/>:</path/in/container/> -d redis
docker run --name redis-1 --rm -v ~/repos/edu-slurm/lecture02/volumes/redis/:/data/ -d redis

# Volumes

https://docs.docker.com/storage/volumes/

docker volume create my-vol
docker volume ls
docker volume inspect my-vol
docker volume rm my-vol

docker run --name redis-2 --rm --mount source=my-vol,target=/app-data -d redis
docker volume inspect my-vol

_None of the containers can share this data if you use the local volume driver._

docker run --name redis-3 --rm --mount source=my-vol,target=/app-data,readonly -d redis

docker run --name redis-2 --rm --mount type=volume,source=my-vol,target=/app-data -d redis

## shared volumes

TBD

## clean up

```
docker volume prune
```
