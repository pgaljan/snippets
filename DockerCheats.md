##### reasonably complete docker run command
```docker
docker run [-p<40443>:<443>] [[-d] | <-it>]] [--rm] [--name nginx] <[imageID] | [nginx:default]>
```
##### reasonably complete docker build command
```docker
docker build -t nginx:default <path/with/Dockerfile>
```
##### run an image
```docker
docker run -p [port]:[port] <image>
```
##### list images
```docker
docker images
```
##### build an image
```docker
docker build .
```
##### pull an image
```docker
docker pull [nginx, etc]
```
##### stop a container
```docker
docker stop [name]
```
##### list containers
```docker
docker ps
```
##### list all containers
```docker
docker ps -a
```
##### interative shell on container
```docker
docker run -it <container>
```
```docker
docker start -ai <container>
```

##### Inspect an image
```docker
docker image inspect
```

##### retrieve logs from container
```docker
docker cp [container]:/path/to/file [localpath]
```
##### Prune non-running images
*-a to remove tagged images*
```docker
docker image prune
```
##### Remove images selectively
```docker
docker rmi <image> [image] ...
```
##### Prune non-running containers

```docker
docker container prune
```

##### Remove containers selectively
```docker
docker rm <container> [container] ...
```
##### Name a container
```docker
docker run --name <string>
```
##### Name an image
```docker
docker build -t <name>:[tag]
```