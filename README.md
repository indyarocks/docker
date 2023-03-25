# Docker Notes:

## Docker Hub:
https://hub.docker.com/

## What is container
1. Layers of images
2. Mostly Linux Base image, because small in size (usually Alpine)
3. Application image on top

## Pulling the application image
1. From docker hub, search for the image
2. Pull the image:
 - docker pull postgres:<version> OR
 - docker run postgres:<version - 14.7>
```
Error: Database is uninitialized and superuser password is not specified.
       You must specify POSTGRES_PASSWORD to a non-empty value for the
       superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".

       You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
       connections without a password. This is *not* recommended.

       See PostgreSQL documentation about "trust":
       https://www.postgresql.org/docs/current/auth-trust.html
```
- docker run --name chandan-mysql-9.0.32 -e MYSQL_ROOT_PASSWORD=password -d mysql:8.0.32
- 
## Docker commands
1. `docker ps` - To get list of running containers
2. `docker pull <image-name-from-docker-hub>` - To pull latest image for the app
3. `docker images` - To get the list of local images
4. `docker run redis` - To run container of latest image of redis in foreground
5. `docker run redis -d` -> To run the image container in background/detached mode
6. `docker stop <container-id>` -> To stop the container, get container id using `docker ps`
7. `docker start <container-id>` -> To start the stopped container, use  `docker ps -a` to get the container id
8. `docker ps -a` -> To list all containers (running and stopped both)
9. `docker run redis:7.0.10` -> To run redis 7.0.10 version
10. `docker logs <container-id/container_name>` To check the logs for a container either container id or container name
11. `docker run -p6000:6379 -d redis` To bind host port 6000 to container port 6379 in detached mode
12. `docker run -d -p6001:6379 --name redis-6 redis:6` To run redis version 6 in detached mode with host port 6001 and name `redis-6`
13. `docker exec -it <container-id/container-name> /bin/bash` -> To start `interactive terminal` for a given container id OR name
14. 


## Docker container PORTS
```shell
chandan@~/Workspace/2023/docker (main) ± ➜ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS      NAMES
322df54a488a   redis:7.0.10   "docker-entrypoint.s…"   5 seconds ago   Up 5 seconds   6379/tcp   reverent_kare

chandan@~/Workspace/2023/docker (main) ± ➜ docker run -d redis
4a13df3664307d76f2e8c081167b3c1ad3ca54a2c3d7ac445ba0db7fcd7c536b

chandan@~/Workspace/2023/docker (main) ± ➜ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS      NAMES
4a13df366430   redis          "docker-entrypoint.s…"   4 seconds ago    Up 3 seconds    6379/tcp   compassionate_swartz
322df54a488a   redis:7.0.10   "docker-entrypoint.s…"   32 seconds ago   Up 31 seconds   6379/tcp   reverent_kare
```
Both the containers are running on 6379/tcp port.
![alt text](images/container-port-vs-host-port.png "Container-Port")

We need to bind the container port to host (local machine) port. If we open two container port to
same host port, we'll get error. Container port can be same for two or more containers, but they 
must be bound to different host port.

To bind the port to specific host port, we need to provide host port while running/starting the container.
```shell
chandan@~/Workspace/2023/docker (main) ± ➜ docker stop 322df54a488a
322df54a488a

chandan@~/Workspace/2023/docker (main) ± ➜ docker stop 4a13df366430
4a13df366430

chandan@~/Workspace/2023/docker (main) ± ➜ docker run -p6000:6379 -d redis
72b2018cfe57145766c052cb1167134a87c3608b591d7c6d68ad51ca16e0db6b

chandan@~/Workspace/2023/docker (main) ± ➜ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                    NAMES
72b2018cfe57   redis     "docker-entrypoint.s…"   4 seconds ago   Up 3 seconds   0.0.0.0:6000->6379/tcp   ecstatic_chandrasekhar

chandan@~/Workspace/2023/docker (main) ± ➜ docker run -p3000:6379 -d redis:7.0.10
958e01ceb5424c81e41298a599c20be7e4f02e40564ebadb4416a466e5e0defc

chandan@~/Workspace/2023/docker (main) ± ➜ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                    NAMES
958e01ceb542   redis:7.0.10   "docker-entrypoint.s…"   3 seconds ago    Up 2 seconds    0.0.0.0:3000->6379/tcp   inspiring_hermann
72b2018cfe57   redis          "docker-entrypoint.s…"   35 seconds ago   Up 35 seconds   0.0.0.0:6000->6379/tcp   ecstatic_chandrasekhar

```
Now the host port 6000 is listening to latest redis container. Host port 3000 is listening to redis 7.0.10 docker container.

## Important concepts
1. Docker image and container : Docker image is actual package of the application, however, container is actually starting the application.
2. `docker run -d -p --name redis-latest redis:6` Docker run command have few options. It runs the image. However `docker start <container-id>` simply start the container based on the initial configuration given during `docker run`

## Docker debugging
`docker exec -it <container-id/container-name> /bin/bash` -> To start `interactive terminal` for a given container id OR name
```shell
chandan@~/Workspace/2023/docker (main) ± ➜ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                    NAMES
a68f4e2a458d   redis          "docker-entrypoint.s…"   4 seconds ago    Up 3 seconds    0.0.0.0:6000->6379/tcp   redis-latest
1b28812476ee   redis:6        "docker-entrypoint.s…"   2 minutes ago    Up 2 minutes    0.0.0.0:6001->6379/tcp   redis-6
958e01ceb542   redis:7.0.10   "docker-entrypoint.s…"   31 minutes ago   Up 31 minutes   0.0.0.0:3000->6379/tcp   inspiring_hermann

chandan@~/Workspace/2023/docker (main) ± ➜ docker exec -it a68f4e2a458d /bin/bash
root@a68f4e2a458d:/data# pwd
/data
root@a68f4e2a458d:/data# ls
root@a68f4e2a458d:/data# cd /
root@a68f4e2a458d:/# ls
bin  boot  data  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@a68f4e2a458d:/# 
root@a68f4e2a458d:/# env
HOSTNAME=a68f4e2a458d
REDIS_DOWNLOAD_SHA=1dee4c6487341cae7bd6432ff7590906522215a061fdef87c7d040a0cb600131
PWD=/
HOME=/root
REDIS_VERSION=7.0.10
GOSU_VERSION=1.16
TERM=xterm
REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-7.0.10.tar.gz
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
OLDPWD=/data
root@a68f4e2a458d:/# exit
exit

```
Since most of the container images are based on light-weight linux distribution, so we may not have all shell commands
available in this `interactive terminal` mode. Example below:

```shell
chandan@~/Workspace/2023/docker (main) ± ➜ docker exec -it a68f4e2a458d /bin/bash
root@a68f4e2a458d:/data# ls  
root@a68f4e2a458d:/data# cd /
root@a68f4e2a458d:/# ls
bin  boot  data  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@a68f4e2a458d:/# curl
bash: curl: command not found

```
## Docker playground
https://labs.play-with-docker.com/

## Docker Learning Resources
1. Learning Docker - https://youtu.be/3c-iBn73dDE
2. Series - https://youtu.be/GeqaTjKMWeY

