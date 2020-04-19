```
docker-compose up -d
```

OR force building images, even existing

```
docker-compose up --build -d
```

```
docker-compose ps

docker exec -it py-example_redis_1 /bin/sh
docker exec -it py-example_web_1 /bin/sh
docker inspect py-example_redis_1 | grep "Memory\|NanoCpus"

docker-compose down
```
