### Lets use docker

So you want to know docker, this is a zero to basic functionality post. I am not going to go into depth of really anything and just give you how to build a container and then how to make actually work with it on your machine.

### Needed

not going to go over how to install these, but just have them installed

- docker
- docker-compose

### Basic what is a docker?

A simple way to think of a docker is like a lighter weight VM, and that is about all we are going to care about in this post.
Terminology

- layer - The building blocks of an image. A layer represent an instruction in a Dockerfile and are the stored result of running that instruction against an existing image. Typically read only unless it is the last layer used to make a container.
- image - A series of layers stacked on top of each other
- container - An image with a writable layer stacked onto the very end of it. Basically an entire deployment of an OS and application (but not really).

#### Image here

- Dockerfile - a series of instructions that define the layers and is used to build an image
- docker-compose.yml - a yaml file used to define how to build images and deploy a series of docker containers

### So how do I build a container?

The environment that we are going to use for this post is as follows, one directory with three files within it. All of our commands are executed within the application directory.

```
/application/Dockerfile
/application/docker-compose.yml
/application/hello_world.py
```

```
Dockerfile

FROM ubuntu
RUN apt-get update && apt-get install -y python
COPY . /app
CMD python /app/hello_world.py
```

```
hello_world.py

print("hello world")
```

To use docker we should start with a

### Dockerfile

A Dockerfile really contains a list of commands that define the environment you intend to build. Each line is a single layer and those layers stacked on top off each other is an image.

```
FROM ubuntu
RUN apt-get update && apt-get install -y python
COPY . /app
CMD python /app/hello_world.py
```

This is a simple dockerfile that makes use of the four commands

- FROM - defines a base image for subsequent layers to build off of. For now we will just use this to define what OS we want our container to be. That image will be pulled into your dockers images.
- RUN - runs a command using /bin/sh
- COPY - Copies files into the filesystem of the image
- CMD - defines the default command for the image
  So with this file we have defined an environment that is Ubuntu 15.04, has a copy of all the files in the current directory accessable within /app and will run “python /app/hello_world.py” on execution. But currently we only have the environment described, not an actual image

### Building an image from a Dockerfile

So lets build the image. To do so we will use docker build

```
docker image build .
```

A simple run of docker build takes a path argument which it will search for a Dockerfile. If found then the Dockerfile will be used to create an image and example run would look like follows

```
Sending build context to Docker daemon 3.072kB
Step 1/3 : FROM ubuntu
15.04: Pulling from library/ubuntu
9502adfba7f1: Pull complete
4332ffb06e4b: Pull complete
2f937cc07b5f: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:2fb27e433b3ecccea2a14e794875b086711f5d49953ef173d8a03e8707f1510f
Status: Downloaded newer image for ubuntu
Sending build context to Docker daemon 3.072kB
---> 94e814e2efa8
Step 2/4 : RUN apt-get update && apt-get install -y python
---> 11681278e3eb
Step 3/4 : COPY . /app
---> 5039c9bf6766
Step 4/4 : CMD python /app/hello_world.py
---> Running in f25f4e47afb0
Removing intermediate container f25f4e47afb0
---> 7c998da94cfe
Successfully built 7c998da94cfe
```

So what happens is we start with pulling down an end image which is saved as image 94e814e2efa8, we then RUN apt-get update and apt-get install on the ubuntu image to create layer 11681278e3eb, followed by applying COPY on that layer to create the layer 5039c9bf6766, finally we take our CMD to set up the layer to be save as the final image 7c998da94cfe. We can verify this with the following commands.

```
docker image ls

REPOSITORY TAG IMAGE ID CREATED SIZE
<none> <none> 7c998da94cfe 4 minutes ago 148MB
ubuntu latest 94e814e2efa8 3 weeks ago 88.9MB
```

with this command we can see our base image and the end image, but what about the intermediate images? If we do docker image ls -a we will actually see it. Docker ls hides intermediate images by default and -a will show them all.

```
docker images ls -a

REPOSITORY TAG IMAGE ID CREATED SIZE
<none> <none> 7c998da94cfe 26 minutes ago 48MB
<none> <none> 5039c9bf6766 7 minutes ago 148MB
<none> <none> 11681278e3eb 18 hours ago 148MB
ubuntu latest 94e814e2efa8 3 weeks ago 88.9MB
```

But if you have been paying attention I called intermediate images layers before. That is because they are not actually taking up 131MB a piece if we run docker image history <image id> we can see the break down of the image.

```
docker image history 7c998da94cfe

IMAGE CREATED CREATED BY SIZE COMMENT
7c998da94cfe 7 minutes ago /bin/sh -c #(nop) CMD ["/bin/sh" "-c" "pyth… 0B
5039c9bf6766 7 minutes ago /bin/sh -c #(nop) COPY dir:4de93998b3d5f84fb… 124B
11681278e3eb 18 hours ago /bin/sh -c apt-get update && apt-get install… 59.4MB
94e814e2efa8 3 weeks ago /bin/sh -c #(nop) CMD ["/bin/bash"] 0B
<missing> 3 weeks ago /bin/sh -c mkdir -p /run/systemd && echo 'do… 7B
<missing> 3 weeks ago /bin/sh -c rm -rf /var/lib/apt/lists/\* 0B
<missing> 3 weeks ago /bin/sh -c set -xe && echo '#!/bin/sh' > /… 745B
<missing> 3 weeks ago /bin/sh -c #(nop) ADD file:1d7cb45c4e196a6a8… 88.9MB
```

This is our layers for our end image and what they contribute. <missing> are layers that were used to build an image on another machine leading to our starting image 94e814e2efa8 which is sized at 88.9MB, which our RUN adds 59.4MB to, our COPY adds 124B to and our CMD adds 0B. Meaning that all together our images take up 148 MB of space like our docker image ls shows, but the image ls deceptively shows the intermediate images (layers) as the same size.
To clear things up even more let’s tag our image essentially naming it.

```
docker image build -t hello_world .

Sending build context to Docker daemon 3.072kB
Step 1/4 : FROM ubuntu
---> 94e814e2efa8
Step 2/4 : RUN apt-get update && apt-get install -y python
---> Using cache
---> 11681278e3eb
Step 3/4 : COPY . /app
---> Using cache
---> 5039c9bf6766
Step 4/4 : CMD python /app/hello_world.py
---> Using cache
---> 7c998da94cfe
Successfully built 7c998da94cfe
Successfully tagged hello_world:latest

docker image ls -a

hello_world latest 7c998da94cfe 13 minutes ago 148MB
<none> <none> 5039c9bf6766 13 minutes ago 148MB
<none> <none> 11681278e3eb 18 hours ago 148MB
ubuntu latest 94e814e2efa8 3 weeks ago 88.9MB
```

Now we can see hello_world is our final image from our build command, and all of the <none> are stages in between. You can see during the build it mentions Using cache. Docker can detect with some commands that a layer for that command already exists and will use that layer to build the next layer saving both space and build time, but that isn’t a focus of this post.
With the image now named we can interact with it using the name instead of the hash.

```
docker image history hello_world

IMAGE CREATED CREATED BY SIZE COMMENT
7c998da94cfe 16 minutes ago /bin/sh -c #(nop) CMD ["/bin/sh" "-c" "pyth… 0B
5039c9bf6766 16 minutes ago /bin/sh -c #(nop) COPY dir:4de93998b3d5f84fb… 124B
11681278e3eb 18 hours ago /bin/sh -c apt-get update && apt-get install… 59.4MB
94e814e2efa8 3 weeks ago /bin/sh -c #(nop) CMD ["/bin/bash"] 0B
<missing> 3 weeks ago /bin/sh -c mkdir -p /run/systemd && echo 'do… 7B
<missing> 3 weeks ago /bin/sh -c rm -rf /var/lib/apt/lists/\* 0B
<missing> 3 weeks ago /bin/sh -c set -xe && echo '#!/bin/sh' > /… 745B
<missing> 3 weeks ago /bin/sh -c #(nop) ADD file:1d7cb45c4e196a6a8… 88.9MB
```

Let’s clean up this image and move on to running a docker image

```
docker image rm hello_world

Untagged: hello_world:latest
Deleted: sha256:7c998da94cfea0449aa33a6f81a6bd1800d51bf7edbf3abbead66930194380e4
Deleted: sha256:5039c9bf6766aa61e37ffbb1869d2e5817878a81b7f900fc6397dd0818c4db8a
Deleted: sha256:6b9e57b8a5a270b7edee6b2263820bd87033c0f386c5d3beb361340b659fce3b
Deleted: sha256:11681278e3ebd549400378fab5591dc5e313ff1f54d4fbcb0feceaf0c5857b99
Deleted: sha256:6783a49e12d5927cb344d6c6280b27a1b69107baae71e8494dc059d0f15335a0
```

### Running a docker container starting from a Dockerfile.

We will use the same files from earlier to go onto the next step and skip over all of the set up getting our image.
Our directory structure:

```
/application/Dockerfile
/application/docker-compose.yml
/application/hello_world.py
```

Our hello_world.py

```
print("hello world")
```

Our Dockerfile

```
FROM ubuntu
RUN apt-get update && apt-get install -y python
COPY . /app
CMD python /app/hello_world.py
```

The first thing we need is to build our image

```
docker image build -t hello_world .

Sending build context to Docker daemon 3.072kB
Step 1/4 : FROM ubuntu
---> 94e814e2efa8
Step 2/4 : RUN apt-get update && apt-get install -y python
---> Running in b48830c56bc5
...
Removing intermediate container b48830c56bc5
---> 61c089bfd659
Step 3/4 : COPY . /app
---> 322c582ae272
Step 4/4 : CMD python /app/hello_world.py
---> Running in ad295b95c861
Removing intermediate container ad295b95c861
---> 4c508f134d4a
Successfully built 4c508f134d4a
Successfully tagged hello_world:latest
```

With this we are back to the end part of the last section, we have an image named hello_world which we need to run. Which we can now run

```
docker container run hello_world

hello_world
```

Run will take an image and create a new container with that image then run the provided command in that container, if no command is provided then it falls back into the container defined one in this case it is from CMD python /app/hello_world
Let’s find the actual container.

```
docker container ls

CONTAINER ID IMAGE COMMAND CREATED STATUS
```

But there is nothing there, that is because by default our container ls only shows running instances, if we add a -a we will see the container that we just ran.

```
docker container ls -a

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
7899ccf1abfe hello_world "/bin/sh -c 'python …" 8 seconds ago Created boring_lalande
```

Here we can see our container got created with the command “/bin/sh -c ‘python /app/hello_world.py’” to a default name of boring_lalande. Unlike images, containers will get created using a name no matter what. If none is provided it will be randomly generated. But let’s remake our container so that we have a name that we want

```
docker container run --name hello_world hello_world

docker container ls -a

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
4276ddff546e hello_world "/bin/sh -c 'python …" 42 seconds ago Created hello_world
7899ccf1abfe hello_world "/bin/sh -c 'python …" 6 minutes ago Created boring_lalande
```

Now we have a container with the name hello_world, lets clean up the old one real quick.

```
docker container rm boring_lalande

boring_lalande

docker container ls -a

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
4276ddff546e hello_world "/bin/sh -c 'python …" About a minute ago Created hello_world
```

So now that we have a named container lets actually talk about what is actually happening with this container. This container is just another layer, except unlike the ones up to the end of the image, this layer is writable and has the hash 4376ddff546e.... This is the layer we use to actually execute our programs, while each container spun up will use the image as a base and make any modifications only to it’s writable layer not effecting any other containers that may use the same base. This container can actually be converted into an image itself which would take the writable layer, make a copy of it and mark it read only. You shouldn’t do this but it is possible and is being used to explain that containers are really just another layer added to the end of an image.
Now most of the time you are going to use run to interact with images and make containers, but if you have a failed or exited container there are commands you can use to continue to work with it. The hashes have changed from this point since I rebuilt, but because it is named now this does not matter.
We can start our container again using

```
docker container start hello_world

hello_world
```

This container as is expected by the CMD python /app/hello_world.py line declares runs hello_world.py which prints “hello world” and exits. We can look at more information about this container by using some docker container commands.

```
docker container ls -a

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
4cc1106a5cb7 hello_world "/bin/sh -c 'python …" 25 seconds ago Exited (0) 7 seconds ago hello_world

docker container logs hello_world

hello_world
```

With these commands we can see the container was run 7 seconds ago, exited with status code(0), and that the stdout logged hello_world. If we run it a few more times and then check the logs we can see that the log appends new logs onto the end of an existing containers logs.

```
docker container start hello_world
docker container start hello_world
docker container logs hello_world

hello world
hello world
hello world
```

These commands can be useful for working with and debugging a container after it’s lifecycle is complete. Most of the other container commands can be used while a container is running in more long running containers like web servers or daemonic applications.

### Providing more configuration on a run

Applications can have many variables that may change how it runs. Secrets, which you do not want to store within the applications code base or on the image that anyone can pull down. Environment variables that may be used to define that this application is being run in production or that it is running in QA. Quotas to make scheduling easier and to keep the application within expected bounds. These are only relevant on run and not during the build image meaning they will not be and should not be provided during the build of the image but only at the run time. You can provide these and more during the docker run command

```
docker run --help

Options:
--add-host list Add a custom host-to-IP mapping (host:ip)
-a, --attach list Attach to STDIN, STDOUT or STDERR
--blkio-weight uint16 Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
--blkio-weight-device list Block IO weight (relative device weight) (default [])
--cap-add list Add Linux capabilities
--cap-drop list Drop Linux capabilities
--cgroup-parent string Optional parent cgroup for the container
--cidfile string Write the container ID to the file
--cpu-period int Limit CPU CFS (Completely Fair Scheduler) period
--cpu-quota int Limit CPU CFS (Completely Fair Scheduler) quota
--cpu-rt-period int Limit CPU real-time period in microseconds
--cpu-rt-runtime int Limit CPU real-time runtime in microseconds
-c, --cpu-shares int CPU shares (relative weight)
--cpus decimal Number of CPUs
--cpuset-cpus string CPUs in which to allow execution (0-3, 0,1)
--cpuset-mems string MEMs in which to allow execution (0-3, 0,1)
-d, --detach Run container in background and print container ID
--detach-keys string Override the key sequence for detaching a container
--device list Add a host device to the container
--device-cgroup-rule list Add a rule to the cgroup allowed devices list
--device-read-bps list Limit read rate (bytes per second) from a device (default [])
--device-read-iops list Limit read rate (IO per second) from a device (default [])
--device-write-bps list Limit write rate (bytes per second) to a device (default [])
--device-write-iops list Limit write rate (IO per second) to a device (default [])
--disable-content-trust Skip image verification (default true)
--dns list Set custom DNS servers
--dns-option list Set DNS options
--dns-search list Set custom DNS search domains
--domainname string Container NIS domain name
--entrypoint string Overwrite the default ENTRYPOINT of the image
-e, --env list Set environment variables
--env-file list Read in a file of environment variables
--expose list Expose a port or a range of ports
--gpus gpu-request GPU devices to add to the container ('all' to pass all GPUs)
--group-add list Add additional groups to join
--health-cmd string Command to run to check health
--health-interval duration Time between running the check (ms|s|m|h) (default 0s)
--health-retries int Consecutive failures needed to report unhealthy
--health-start-period duration Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s)
--health-timeout duration Maximum time to allow one check to run (ms|s|m|h) (default 0s)
--help Print usage
-h, --hostname string Container host name
--init Run an init inside the container that forwards signals and reaps processes
-i, --interactive Keep STDIN open even if not attached
--ip string IPv4 address (e.g., 172.30.100.104)
--ip6 string IPv6 address (e.g., 2001:db8::33)
--ipc string IPC mode to use
--isolation string Container isolation technology
--kernel-memory bytes Kernel memory limit
-l, --label list Set meta data on a container
--label-file list Read in a line delimited file of labels
--link list Add link to another container
--link-local-ip list Container IPv4/IPv6 link-local addresses
--log-driver string Logging driver for the container
--log-opt list Log driver options
--mac-address string Container MAC address (e.g., 92:d0:c6:0a:29:33)
-m, --memory bytes Memory limit
--memory-reservation bytes Memory soft limit
--memory-swap bytes Swap limit equal to memory plus swap: '-1' to enable unlimited swap
--memory-swappiness int Tune container memory swappiness (0 to 100) (default -1)
--mount mount Attach a filesystem mount to the container
--name string Assign a name to the container
--network network Connect a container to a network
--network-alias list Add network-scoped alias for the container
--no-healthcheck Disable any container-specified HEALTHCHECK
--oom-kill-disable Disable OOM Killer
--oom-score-adj int Tune host's OOM preferences (-1000 to 1000)
--pid string PID namespace to use
--pids-limit int Tune container pids limit (set -1 for unlimited)
--privileged Give extended privileges to this container
-p, --publish list Publish a container's port(s) to the host
-P, --publish-all Publish all exposed ports to random ports
--read-only Mount the container's root filesystem as read only
--restart string Restart policy to apply when a container exits (default "no")
--rm Automatically remove the container when it exits
--runtime string Runtime to use for this container
--security-opt list Security Options
--shm-size bytes Size of /dev/shm
--sig-proxy Proxy received signals to the process (default true)
--stop-signal string Signal to stop a container (default "SIGTERM")
--stop-timeout int Timeout (in seconds) to stop a container
--storage-opt list Storage driver options for the container
--sysctl map Sysctl options (default map[])
--tmpfs list Mount a tmpfs directory
-t, --tty Allocate a pseudo-TTY
--ulimit ulimit Ulimit options (default [])
-u, --user string Username or UID (format: <name|uid>[:<group|gid>])
--userns string User namespace to use
--uts string UTS namespace to use
-v, --volume list Bind mount a volume
--volume-driver string Optional volume driver for the container
--volumes-from list Mount volumes from the specified container(s)
-w, --workdir string Working directory inside the container
```

An example could be done like

```
sudo docker run --memory 4MB --env "environment=PRODUCTION" hello_world
```

### Managing docker with docker-compose

A utility called docker compose allows us to manage multiple containers at once. Using yaml we can declare the environment we want to manage choosing what Dockerfiles and images we want to run in multiple containers. Lets imagine we wanted to set up our hello-world container with the environment variable environment=PRODUCTION and we also wanted a redis server to be deployed along side our hello world container.

```
version: '3'

services:
hello-world:
build: .
environment: - environment=PRODUCTION

     redis:
         image: 'redis:alpine'
```

With this docker-compose.yml file we can then use docker-compose up to build and run both hello-world using the Dockerfile in directory . to build an the environment variable environment=PRODUCTION and another container redis using the image redis:alpine to deploy any run. By default docker-compose up runs attached so to break out we can use ctrl+c. You can also run docker-compose up -d to run it detached leaving your containers running in the background. When using docker-compose you get many of the options available with docker but it now will only apply to your managed containers for instance you can still use logs and ps to view your containers.

```
docker-compose up -d
docker-compose ps

          Name                         Command               State     Ports

---

basicdocker_hello-world_1 /bin/sh -c python /app/hel ... Exit 0
basicdocker_redis_1 docker-entrypoint.sh redis ... Up 6379/tcp

docker-compose logs

hello-world_1 | hello world
redis_1 | 1:C 14 Jun 2019 02:24:43.069 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1 | 1:C 14 Jun 2019 02:24:43.069 # Redis version=5.0.5, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1 | 1:C 14 Jun 2019 02:24:43.069 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_1 | 1:M 14 Jun 2019 02:24:43.071 \* Running mode=standalone, port=6379.
redis_1 | 1:M 14 Jun 2019 02:24:43.071 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_1 | 1:M 14 Jun 2019 02:24:43.071 # Server initialized
```

A word of warning. Your image is only built if an image with the expected name does not exist. If it already exists when working with docker-compose and you make changes to the files or the Dockerfile you will need to run docker-compose build to rebuild the image before running up again.
And with that we are done. From this you should be able to build images, run the images as containers, basic manage images and containers, and use basic docker-compose commands. Quickly here are the basics all over again.

```
docker image build . / docker build .
builds an image from a Dockerfile in .

docker build . -t aname
builds an image with the name aname

docker image ls / docker images
view images on the local machine

docker image history aname / docker history aname
view the breakdown of the layers that build up aname

docker container run aname / docker run aname
takes image a name and makes it a container by putting a writable layer on it and executing the ENTRYPOINT or CMD defined in the image

docker container run --name containername aname
runs aname as a container under the name containername

docker container ls / docker ps
list docker containers (only active ones by default

docker container logs containername / docker logs containername
prints the logs from container containername

docker-compose build
builds images from docker-compose.yml in the current directory

docker-compose up
potentially builds images and then runs them using docker-compose.yml in the current directory

docker-compose ps
view status of containers managed by docker-compose

docker-compose logs
view logs of containers managed by docker-compose
```

Getting a docker file to run

```
docker build . -t myimage
docker run myimage
```
