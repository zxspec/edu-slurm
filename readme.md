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
docker run --rm -p 80:8080 -d nginx
