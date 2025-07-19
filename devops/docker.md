# Docker
A container is a single package containing everything to run an application meaning a system(like Alpine Linux), a programming language runtime(like Python) to run a script or an executable(built with a compiler) that will often provide an API, a frontend(like React) or both
The kernel is shared with containers from the underlying operating system
## Examples
Alpine Linux + Python + SQlite3 + FastAPI + a static HTML, CSS and JS frontend
Debian Linux + a Go executable providing an API with Chi + a React frontend
## OCI(Open Container Initiative)
Bunch of major companies such as Docker, Google and VMware came together to make a standard for containers
This includes: 
* Containers runtime specification, like Docker and Podman
* Containers images specification, like what metadata should a container have to be effectively ran by the runtime
* Distribution specification, think pulling an image and registers
Docker is a specific implementation of this OCI specification and uses the Containerd runtime
### Random though on runtimes
Even though Podman might be better architected, the industry won't move away from Docker for a long time just like happened with Node
# All together now
Kubernetes, Swarm and Nomad from Hashicorp are orchestrators able to manage(meaning create, remove and such) containers on a single computer or an hypervisor with multiple vms(aka nodes)
# Internals
* Namespaces, provides isolation for system resources.
Each Namespace type will isolate a certain type of resources such as PID for processes, USER for users and NETWORK for things like interfaces and  ports while allowing mapping users and ports to the host(running the containers runtime) system resources(e.g. -p/--port).
PID 1 inside a container will be the container's init even though there is obviously another init running on the host.
* Control Groups, a Linux kernel feature(not a single nor a group of syscalls) limiting a process system resources usage
Application 1 running in a container might have maximum of 30% CPU usage and 500MB of memory
This allows to avoid "the noisy neighbour" process which happens when another process takes too many available resources making another unable to run properly
Docker automatically handles cgroups for you!
```sh
docker run -it --cpus="1.5" --memory="512m" image
```
You can also use this by itself because again, it's a Linux Kernel feature:
```sh
# Are you using cgroups?
mount | grep cgroup
# Create a new cgroup
mkdir /sys/fs/cgroup/my_process_group
# Set memory limit to 500MB
echo "500M" | sudo tee /sys/fs/cgroup/my_process_group/memory.max
# Enable memory accounting
echo "1" | sudo tee /sys/fs/cgroup/my_process_group/memory.stat
# Assign a PID to the cgroup
echo "2222" | sudo tee /sys/fs/cgroup/my_process_group/cgroup.procs
# Check the current cgroup memory usage
cat /sys/fs/cgroup/my_process_group/memory.current
# Delete the cgroup
rm -rf /sys/fs/cgroup/my_process_group
```
* Union Filesystems, it layers multiple filesystems together, used when building a Dockerfile
Every RUN, COPY and such will create a new layer, these layers are stacked using overlayfs
The MOUNT Namespace will provide isolation for mount points and can be used with both "Union Filesystems" and Docker Volumes
## Getting to Volumes
* UnionFS provides the layered image files
* The MOUNT namespace makes this appear as root(/) filesystem to container
* Volumes mount host directories outside this structure on requested
## Facts
The Docker Daemon(`systemctl start containerd docker`) has an HTTP API that it's used whenever you use the Docker CLI or Docker Desktop
On a Windows host you can use either the HyperV or WSL2 for containers using the Windows or the Linux kernel
# Docker CLI
## Running
`docker run org/image arguments`
`docker run docker/whalesay cowsay "Hello!"`
`docker run hello-world`, this one is dimostrative you will always go for the longer format
This will pull and spin up a container running the image which can have tags(for variants and versioning)
### Running the Memos container
```sh
docker run -d --name memos --restart unless-stopped -p 5230:5230 -v ./memos:/var/opt/memos neosmemo/memos:stable
```
#### The Docker Compose version
```sh
# docker-compose.yml
version: "3.8"
services:
  memos:
    image: neosmemo/memos:stable
    container_name: memos
    restart: unless-stopped
    ports:
      - "5230:5230"
    volumes:
      - ./memos:/var/opt/memos
	## Additional enviroment variables
    environment:
      - MEMOS_MODE=prod
      - MEMOS_PORT=5230
```
Run this with:
```sh
docker compose up -d
docker compose down
```
Use `docker attach` for running containers
### Running a Postgres instance
```sh
docker run --env POSTGRES_PASSWORD=ass --publish 5432:5432 postgres:15.1-alpine
```
This took a second, getting a Postgres database running without containers take some extra effort and time
# The Docker Hub regestry
[Docker Hub](https://hub.docker.com/) has a lot of public images ranging from bare operating system images and stuff like Nginx and Caddy which are web servers and reverse proxies to applications such as Memos, Pocketbase or a Minecraft Server
# More
You can use Volume which mount a Container's directory on the host filesystem or you can Bind Mount which does the opposite mounting an host directory on the Container, you should use Volumes by default as performance would be lower with Bind Mounts for read(r) writes(w)
-it is --interactive plus --tty
--rm will remove the container once stopped
## Data Persistancy
Data is persistent in containers only if built into an image when creating them
### Example
```sh
docker run --interactive --tty --rm ubuntu:22.04
```
Now I have a shell in the container, if I then:
```sh
apt update
apt install iputils-ping -y
ping -c 3 google.com
```
Stop the container that gets deleted and make another one, the ping utility won't be there!
But if I take the base image of ubuntu:22.04 and make a Dockerfile which adds ping to it, it will be there for every container of that image
```dockerfile
# Dockerfile
FROM ubuntu:22.04
RUN apt update
RUN apt install iputils-ping -y
```
Now I will build this Docker image, run it and ping will work
```sh
docker built -t ubuntu-ping . # or use the modern BuildKit plugin docker buildx build -t ubuntu-ping .
docker run -it --rm ubuntu-ping
ping -c 3 google.com
```
This will work with a warning(apt isn't a stable interface, because it isn't)
#### Oneline
```sh
docker build --tag ubuntu-ping:revenge -<<EOF
FROM ubuntu:22.04
RUN apt update
RUN apt install iputils-ping -y
EOF
```sh
docker image ls
```
```
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
ubuntu-ping   revenge   5c446e887296   11 seconds ago   145MB
ubuntu-ping   latest    057507d0a0d7   15 minutes ago   145MB
ubuntu        22.04     1d3ca894b30c   4 days ago       77.9MB
```
# Docker Volumes
Create one:
```sh
docker volume create stuff
```
Use it:
```sh
# This obviously only works if stuff exists
docker run -it --rm --mount source=stuff,destination=/data alpine:latest
```
You can now access /data from the alpine container and everything there will be saved to the host filesystem using the stuff volume
## Trick
This will give me a shell into the Docker virtual machine which has the volume data
```sh
# Special public Docker image
docker run -it --rm --privileged --pid=host justincormack/nsenter1@sha256:5af0be5e42ebd55eea2c593e4622f810065c3f45bb805eaacf43f08f3d06ffd8
```
Accessing that data
```sh
cd /var/lib/docker/volumes
ls _data/stuff
```
## Bound Mount
Mo♠unting ./my-data in containers's /data
```sh
# Using $PWD because the source path has to be absolute for a bind mount, an alternative could be $(pwd)
docker run -it --rm --mount type=bind,source=$PWD/my-data,destination=/data ubuntu:22.04
```
Useful when you have something like a config file required for every container of a certain image to run the way you need it to, but data should usually be permenently stored using volumes for better performance
## Shorthand
When using -v/--volume, you will implicitly use a Bind Mount if the host path is absolute and a volume if relative
```sh
# Using Bind Mount for a config file
docker run -d --rm \
-v mongodata:/data/db \
-v ${PWD}/mongod.conf:/etc/mongod.conf \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=ass \
-p 1122:27017 \
mongo:6.0.4 --config /etc/mongod.conf
```
# Getting a specific runtime REPL
```sh
docker run -it --rm python:3.10
```