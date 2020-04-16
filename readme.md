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
