# Configure remote docker context (insecure option)

Using Ubuntu 18.04 + Docker installed with Snappy

## Changes in daemon.json

Instead of `/etc/docker/daemon.json`, Snappy loads config from `/var/snap/docker/current/config/daemon.json`.

Add `hosts` to config
```
{
    . . . 
    
    "hosts": [
        "tcp://192.168.1.30:2375",
        "unix://"
    ]
}


```
Snappy starts `dockerd` with flag: `-H unix://`, in config file using `"unix://"` instead of `fd://`

Restart docker service and test
```
$ sudo snap restart docker.dockerd
$ docker run hello-world

```
In case of configuration errors you would see:
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
Check docker logs
```
$ sudo snap logs docker.dockerd
```

## Client machine
Check connectivity
```
$ curl http://192.168.1.30:2375/containers/json
. . .
[]
```
create context
```
$ docker context create insecure-30 --docker "host=tcp://192.168.1.30:2375"
$ docker context ls
. . .
NAME                DESCRIPTION                               DOCKER ENDPOINT               KUBERNETES ENDPOINT   ORCHESTRATOR
default             Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                         swarm
insecure-30                                                   tcp://192.168.1.30:2375

$ docker context use insecure-30
. . .
insecure-30
Current context is now "insecure-30"

$ docker version
. . .
Client: Docker Engine - Community <--- Local docker
 Version:           19.03.6
 API version:       1.39 (downgraded from 1.40)
 Go version:        go1.12.16
 Git commit:        369ce74a3c
 Built:             Thu Feb 13 01:27:49 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: <--- Remote docker
 Engine:
  Version:          18.09.9
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.13.4
  Git commit:       9552f2b
  Built:            Sat Nov 16 01:07:48 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```


### Links
https://forums.docker.com/t/failed-to-load-listeners-no-sockets-found-via-socket-activation-make-sure-the-service-was-started-by-systemd/62505

