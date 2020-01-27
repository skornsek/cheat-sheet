# Docker 

## Docker basics

### Docker _run_
Detached teče v ozadju flag -d
```
docker run -d jpetazzo/clock    
```
Interaktivno lahko pišeš ukaze v kontejner flag -it

- What does -it stand for?
    - t means "allocate a terminal."
    - i means "connect stdin to the terminal."
- Izhod z ^C in ugasneš kontejner 
- z ^P^Q pošlješ kontejner v detatch mode
```
docker run -it jpetazzo/clock    
```
Custon detatch tipke 
```
docker run -ti --detach-keys ctrl-x,x jpetazzo/clock 
```
Če imaš detatchan kontejner in bi se rat povezal nazaj
```
docker attach <containerID> 
docker attach $(docker ps -lq) // Zadnji pognan 
```
Dodan flag --name poimenujemo instanco _ura_

```
docker run -it --name ura jpetazzo/clock    
```
Restart kontejnerja
```
docker start <yourContainerID> 
```


### Docker _images_
Pregled vseh docker slik 
```
docker images
docker image ls
```
Podatki o specifični sliki
```
docker images just-scratch 
```
Pregled vseh zagnanih docker kontejnerjev
```
docker ps
```
Pregled vseh zagnanih in ugasnjenih docker kontejnerjev
```
docker ps -a
```
Zadnji zagnan kontejner
```
docker ps -l
docker ps -ql // vrne samo id
```
Filtriranje po labeli
```
docker ps --filter label=owner 
```
Brisanje kontejnerjev prisilno ugasne in izbriše
```
 docker rm -f linux_tweet_app
```
Brisanje slik
```
 docker rmi linux_tweet_app
```

### Docker _logs_
Pogledamo loge
```
docker logs <containerID> 
docker logs $(docker ps -ql)  // Logi zadnjega zagnanega
```
Spremljamo loge v živo
```
docker logs --tail 1 --follow <containerID> 
docker logs --tail 1 --follow $(docker ps -ql)  // Logi zadnjega zagnanega
```

### Ustavitev kontejnerjev
Ustavi po 10 sek
```
docker stop <containerID> 
```
Nemudoma ubije
```
docker kill <containerID> 
```

### Docker build 
Zgradi image iz Dockerfile v trenutnem kontekstu z -t poimenujemo image _just-scratch_
```
docker build -t just-scratch .
```

### Docker _exec_
Se uporablja za izvajanje ukazov v kontejnerju, imamo kontejner v detach mode in hočemo v njem izvesti ukaz 

```
docker exec -it ubuntuContainer ls -l  // Zlistamo vsebino direktorija
docker exec -it ubuntuContainer sh // poženemo lupino v že delujočem kontejnerju
```

### Ostalo 
Docker tag poimenuješ image
```
docker tag <newImageId> imeSlike 
```
Pregled kontejnerja v JSON obliki
```
docker inspect <containerID> //contanerjev json opis
docker inspect --format '{{ json .Created }}' <containerID> //primer kako dobimo datum
```
Labele
```
docker run -d -l owner=AMA jpetazzo/clock                                   // Dodajanje
docker inspect $(docker ps -q) --format 'OWNER={{.Config.Labels.owner}}'    // Pregled label
```
Prenos datoteke iz kontejnerja na hosta
```
docker cp <container_id>:/PotDoDatoteke /KamShraneDatoteko 

// Primer prenosa error.loga nginx strežnika v trenutn direktori
docker cp nginx:/var/log/nginx/error.log . 
```

### Networking
Port forwarding
```
docker run -d -P nginx  // Publish all exposed ports to random ports

docker run -d -p 4567:80 nginx //manual port expose format host:container

docker run -d -p 8080:80 -p 8888:80 nginx // exposed on multiple ports
```
kontejner na hostu dostopen na http://localhost:4567/

IP od kontejnerja
```
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <yourContainerID> 

// you can ping it 
```
The built-in drivers include:

- `bridge` _(default)_

- `none` _It can't send or receive network traffic_

- `host` _Performance = native!_

- `container` _It re-uses the network stack of another container._
```
docker run --net bridge|none|host|container:id ...
```

### Communication between containers

Ustvarjanje networka
```
docker network create dev 
```
Izpis omrežij
```
docker network ls 
```
Kreiranje novega kontejnerja z imenom _es_ na omrežju _dev_
```
docker run -d --name es --net dev elasticsearch:2
```
Kreiranje novega linux kontejnerja na omrežju _dev_ z odprto lupino
```
docker run -ti --net dev alpine sh
```
Iz katere lahko direktno izvedemo ping na kontejner z imenom _es_


```
/ # ping es
PING es (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.288 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.070 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.150 ms
```
POZOR: _Names can be resolved only when containers are on the same network._

Net aliasi so izolirani na omrežje in so nedovisni od pravega imena kontejnerja
```
docker network create prod
// We can now create multiple containers with the es alias on the new prod network.
docker run -d --name prod-es-1 --net-alias es --net prod elasticsearch:2
docker run -d --name prod-es-2 --net-alias es --net prod elasticsearch:2
```
```
docker run --net prod --rm alpine nslookup es
Name:      es
Address 1: 172.23.0.3 prod-es-2.prod
Address 2: 172.23.0.2 prod-es-1.prod
```
### Volumes oziroma deljenje diska, map...
- Volumes act as passthroughs to the host filesystem
- Volumes can be created without a container, then used in multiple containers.
- Volumes are not anchored to a specific path.

Zlista vse volume (tudi če kontejner zbrisan)
```
docker volume ls
```
Kreiranje volume
```
docker volume create logs
```
Brisanje volume 
```
docker volume rm logs
```
Povezava host volume do poti v kontejnerju
```
docker run -d -p 1234:8080 -v logs:/usr/local/tomcat/logs tomcat
```
Povezava mape iz hosta v kontejner
```
docker run -d -v /path/on/the/host:/path/in/container image ...

docker run -d -v $(pwd):/src -P namer //doda trenuten direktori kot volume
```

### Omejitve kontejnerjev
Omejitev porabe rama in swapa
```
docker run -ti --memory 100m python 
docker run -ti --memory 100m --memory-swap 100m python
```
Limiting CPU usage
- setting a relative priority with `--cpu-shares`,
    - Default 1024
- setting a CPU% limit with `--cpus`,
    - `--cpus 0.1` means 10% of one CPU,
    - `--cpus 1.0` means 100% of one whole CPU,
    - `--cpus 10.0` means 10 entire CPUs.
- pinning a container to specific CPUs with `--cpuset-cpus`.
    - `--cpuset-cpus 0` forces the container to run on CPU 0;
    - `--cpuset-cpus 3,5,7` restricts the container to CPUs 3, 5, 7;
    - `--cpuset-cpus 0-3,8-11` restricts the container to CPUs 0, 1, 2, 3, 8, 9, 10, 11.


## Dockerfiles

- `RUN` ukaz dve oblike: 
    - plain string: `RUN apt-get install figlet`
        - requires /bin/sh to exist in the container
    - JSON: `RUN ["apt-get", "install", "figlet"]`
        - doesn't require /bin/sh to exist in the container
- `CMD` defines a default command to run when none is given. Each CMD will replace and override the previous one.
    ```Docker
    FROM ubuntu
    RUN apt-get update
    RUN ["apt-get", "install", "figlet"]
    CMD figlet -f script hello
    ```
- `ENTRYPOINT` defines a base command (and its parameters) for the container. The command line arguments are appended to those parameters
    ```Docker
    FROM ubuntu
    RUN apt-get update
    RUN ["apt-get", "install", "figlet"]
    ENTRYPOINT ["figlet", "-f", "script"]
    ```
    ```
    $ docker run figlet salut
               _            
              | |           
     ,   __,  | |       _|_ 
    / \_/  |  |/  |   |  |  
     \/ \_/|_/|__/ \_/|_/|_/
    ```
- `CMD` and `ENTRYPOINT` together CMD passes default to entrypoint but if we write the argument we can still override it
    ```Docker
    FROM ubuntu
    RUN apt-get update
    RUN ["apt-get", "install", "figlet"]
    ENTRYPOINT ["figlet", "-f", "script"]
    CMD ["hello world"]
    ```
    ```
    $ docker run myfiglet hola mundo
     _           _                                               
    | |         | |                                      |       
    | |     __  | |  __,     _  _  _           _  _    __|   __  
    |/ \   /  \_|/  /  |    / |/ |/ |  |   |  / |/ |  /  |  /  \_
    |   |_/\__/ |__/\_/|_/    |  |  |_/ \_/|_/  |  |_/\_/|_/\__/
    ```
- `COPY` use COPY to place the source file into the container.
    `COPY /hostFilePath /ContainerPathTo`
    ```Docker
    FROM ubuntu
    RUN apt-get update
    RUN apt-get install -y build-essential
    COPY hello.c /
    RUN make hello
    CMD /hello
    ```


## Scratch kontejnerji 
### Kontejnerji, ki poganjajo binary datoteke
Download binary datoteke hello in dodajanje pravic za poganjanje datoteke
```
curl https://raw.githubusercontent.com/docker-library/hello-world/master/amd64/hello-world/hello -o hello

chmod +x hello 
```
Dockerfile minimalnega kontejnerja, ki poganja binary file
```Docker
FROM scratch
COPY hello /
CMD ["/hello"]
```
Build kontejnerja hello

```
docker build -t hello .
```


### Distorless docker images

[Google distroless images](https://github.com/GoogleContainerTools/distroless)

"Distroless" images contain only your application and its runtime dependencies. They do not contain package managers, shells or any other programs you would expect to find in a standard Linux distribution (hence the name: Distroless :).

Advantages:
- restricting what's in your runtime container to precisely what's necessary for your app
- reducing the attack surface (less tools/libs/apps inside container==less attack surface)
- improves signal to noise ration for security/vulnerability scanners

hello.c file
```c
int main () {
  puts("Hello, world!");
  return 0;
}
```
Dockerfile of a distroless container container size is 18.5Mb

Multi-stage build
```Docker
FROM ubuntu AS build
RUN apt-get update
RUN apt-get install -y build-essential
COPY hello.c /
RUN make hello
FROM  gcr.io/distroless/cc
COPY --from=build /hello /hello
CMD ["/hello"]
```
```
docker build -t hello .
```

### Reducing image size primer:
```Docker
FROM ubuntu
RUN apt-get update \
 && apt-get install xxx \
 && ... \
 && apt-get remove xxx \
 && ...
 ```

### Exploring a crashed container
 ```
docker commit <container_id> debugimage //naredi novo sliko

docker run -ti --entrypoint sh debugimage //custom entrypoint


docker export <container_id> | tar tv //za celoten container dump
 ```

 ## Docker compose
 A Compose file has multiple sections:

- `version` is mandatory. (We should use "2" or later; version 1 is deprecated.)
    - Version 1 is legacy and shouldn't be used.
    (If you see a Compose file without version and services, it's a legacy v1 file.)
    - Version 2 added support for networks and volumes.
    - Version 3 added support for deployment options (scaling, rolling updates, etc).
- `services` is mandatory. A service is one or more replicas of the same image running as containers.

- `networks` is optional and indicates to which networks containers should be connected.
(By default, containers will be connected on a private, per-compose-file network.)

- `volumes` is optional and can define volumes to be used and/or shared by the containers.

Each service in the YAML file must contain either build, or image.

- `build` indicates a path containing a Dockerfile.

- `image` indicates an image name (local, or on a registry).

- If both are specified, an image will be built from the build directory and named image.

- `command` indicates what to run (like CMD in a Dockerfile).

- `ports` translates to one (or multiple) -p options to map ports.
You can specify local ports (i.e. x:y to expose public port x).

- `volumes` translates to one (or multiple) -v options.
You can use relative paths here.
### Docker compose primer1:
```yml
version: "2"
services:
  www:
    build: www
    ports:
      - 8000:5000
    user: nobody
    environment:
      DEBUG: 1
    command: python counter.py
    volumes:
      - ./www:/src
  redis:
    image: redis
```
### Ukazi
```
docker-compose up --build

docker-compose up -d // v ozadju

docker-compose ps // statusi docker-compose

docker-compose kill // ustavi vse

docker-compose rm -f //odsrani vse

docker-compose down // will stop and remove containers
```