##### reasonably complete docker run command
```docker
docker run \
    [-p<40443>:<443>] # Sets port
    [[-d] | [-it]] # Run detached or interactive
    [--rm] # remove when container is stopped
    [--name nginx] # name the container
 # Volumes
     [-v] <container/path> # create an anonymous volume (can also be done in Dockerfile)
     [-v <name>:</container/path>] # create a named volume 
     [-v] </path/to/host>:</container/path>] #bound volume (absolute)
     [-v] < "%cd%":/container/path> # bound volume (workdir Windows)
     [-v] $(pwd):</container/path> # bound volume (workdir Mac)
      [-v] < "%cd%":/container/path:ro:> # read-only bound volume (workdir Windows)
 <[imageID] | [name:tag]> # select the image
```
##### reasonably complete docker build command
```docker
docker build 
 [-t nginx:default] # name:tag the image
 <path/to/folder/> # path with dockerfile
```
##### reasonably complete docker start command
```docker
docker start 
 <[containerID] | [containerName]> # select the container
 [-a] # attach stdout
 [-i] # attach stdin
 ```
##### docker exec 
```docker
docker exec
 [[-d] | [-it]] # Run detached or interactive
 [privileged] # give privileges to the command
 [-w] # set working directory
 [u] # set user <name|uid>[:<group|gid>]
 [command] # command to run 
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
##### cat logs from container
```docker
docker logs <container>
```

##### retrieve files from container
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
docker build -t <name>[:tag]
```
##### named
docker 