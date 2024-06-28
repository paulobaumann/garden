---
title: Docker
date: 2022-05-13
---

Notes from the course *Docker for absolute beginners* presented by Saurabh
Dhingra on [Coursera](https://www.coursera.org/learn/docker-for-absolute-beginners/).

Why Docker? The different components of an application may have specific
requirements regarding libraries and dependencies, operating system, and server.
And those requirements might be imcompatible.

One could install them on different VMs, but that is costly. Alternative:
putting them on containers.

Container is an isolated area with its on dependencies and libraries installed.
The containers run on the same OS and the same server.

docker is installed as a service. On Ubuntu, to check if it is running:

    systemctl status docker

If the application running on a docker crashes, we can run it without using
systemctl, using `sudo dockerd`. It shows log messages. Or even `sudo dockerd --debug`.

How to create and run a docker container from a docker image. Docker image
contains application code and stuff to make the application run. Example (after
having called `sudo dockerd` in a separate shell):

    rhyme@ip-10-199-122-109:~$ docker run hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    2db29710123e: Pull complete 
    Digest: sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17
    Status: Downloaded newer image for hello-world:latest

    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
     A. The Docker client contacted the Docker daemon.
     B. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
     C. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     D. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
     https://hub.docker.com/

    For more examples and ideas, visit:
     https://docs.docker.com/get-started/

## The docker image

There is a command in each docker image which initiates the docker execution. In
this `hello-world` image, it has printed that previous message.

It gets the image from the the [docker hub](hub.docker.com). That is the
default docker public registry.

There is no container running now:

    rhyme@ip-10-199-122-109:~$ docker container ps
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

The history of execution of the containers shows it has exited:

    rhyme@ip-10-199-122-109:~$ docker ps -a 
    CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
    771570b2ef26   hello-world   "/hello"   6 minutes ago   Exited (0) 6 minutes ago             awesome_galileo

This container had the name `awesome_galileo`. Each time a image is executed, a
container with a new name is created. One can specify the container name using
the option `--name`. On this image, it was executed the command `/hello`.

Some of the flags used for running a image are:

* `i`: interactive (keep STDIN open even if not attached)
* `t`: allocate a pseudo-TTY
* `d`: detach (run container in background and print container ID)

Running ubuntu as an image:

    rhyme@ip-10-199-122-109:~$ docker run -itd ubuntu
    Unable to find image 'ubuntu:latest' locally
    latest: Pulling from library/ubuntu
    125a6e411906: Pull complete 
    Digest: sha256:26c68657ccce2cb0a31b330cb0be2b5e108d467f641c62e13ab40cbec258c68d
    Status: Downloaded newer image for ubuntu:latest
    5d0c8bb99ebdf5a023983f6559cae6ee8798f4c6d8fb2a00f09a2b1b1de5e92f

    rhyme@ip-10-199-122-109:~$ docker ps
    CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
    5d0c8bb99ebd   ubuntu    "bash"    3 minutes ago   Up 3 minutes             gallant_chebyshev

To stop the container, either give its name or its ID (can be only some chars of
it). It can be restarted given the same identification:

    rhyme@ip-10-199-122-109:~$ docker stop 5d
    5d

    rhyme@ip-10-199-122-109:~$ docker ps -a
    CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                       PORTS     NAMES
    5d0c8bb99ebd   ubuntu        "bash"     6 minutes ago    Exited (137) 8 seconds ago             gallant_chebyshev
    fcf827ec1e62   hello-world   "/hello"   9 minutes ago    Exited (0) 9 minutes ago               eloquent_ramanujan
    771570b2ef26   hello-world   "/hello"   20 minutes ago   Exited (0) 20 minutes ago              awesome_galileo

    rhyme@ip-10-199-122-109:~$ docker start 5d
    5d

    rhyme@ip-10-199-122-109:~$ docker ps
    CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
    5d0c8bb99ebd   ubuntu    "bash"    7 minutes ago   Up 8 seconds             gallant_chebyshev

To remove a container:

    rhyme@ip-10-199-122-109:~$ docker ps -a
    CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                        PORTS     NAMES
    5d0c8bb99ebd   ubuntu        "bash"     13 minutes ago   Exited (137) 23 seconds ago             gallant_chebyshev
    fcf827ec1e62   hello-world   "/hello"   16 minutes ago   Exited (0) 16 minutes ago               eloquent_ramanujan
    771570b2ef26   hello-world   "/hello"   26 minutes ago   Exited (0) 26 minutes ago               awesome_galileo

    rhyme@ip-10-199-122-109:~$ docker rm 5d
    5d

    rhyme@ip-10-199-122-109:~$ docker ps -a
    CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
    fcf827ec1e62   hello-world   "/hello"   16 minutes ago   Exited (0) 16 minutes ago             eloquent_ramanujan
    771570b2ef26   hello-world   "/hello"   26 minutes ago   Exited (0) 26 minutes ago             awesome_galileo

Remove all containers:

    rhyme@ip-10-199-122-109:~$ docker container prune
    WARNING! This will remove all stopped containers.
    Are you sure you want to continue? [y/N] y
    Deleted Containers:
    fcf827ec1e6236b54eec64f5d7d62e4f89155c01ac420d063821461e0b406298
    771570b2ef26296374dbb7eea070d34a4400e0da5de8da7312263aa0870c5953

    Total reclaimed space: 0B
    rhyme@ip-10-199-122-109:~$ docker ps -a
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

To remove images (it will only affect images that are not used by/attached to
any container):

    rhyme@ip-10-199-122-109:~$ docker images
    REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
    ubuntu        latest    d2e4e1f51132   13 days ago    77.8MB
    hello-world   latest    feb5d9fea6a5   7 months ago   13.3kB

    rhyme@ip-10-199-122-109:~$ docker image ls
    REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
    ubuntu        latest    d2e4e1f51132   13 days ago    77.8MB
    hello-world   latest    feb5d9fea6a5   7 months ago   13.3kB

    rhyme@ip-10-199-122-109:~$ docker rmi ubuntu
    Untagged: ubuntu:latest
    Untagged: ubuntu@sha256:26c68657ccce2cb0a31b330cb0be2b5e108d467f641c62e13ab40cbec258c68d
    Deleted: sha256:d2e4e1f511320dfb2d0baff2468fcf0526998b73fe10c8890b4684bb7ef8290f
    Deleted: sha256:e59fc94956120a6c7629f085027578e6357b48061d45714107e79f04a81a6f0c

    rhyme@ip-10-199-122-109:~$ docker image rm hello-world
    Untagged: hello-world:latest
    Untagged: hello-world@sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17
    Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
    Deleted: sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359

    rhyme@ip-10-199-122-109:~$ docker image prune
    WARNING! This will remove all dangling images.
    Are you sure you want to continue? [y/N] y
    Total reclaimed space: 0B

To download an image without running it:

    rhyme@ip-10-199-122-109:~$ docker pull ubuntu
    Using default tag: latest
    latest: Pulling from library/ubuntu
    125a6e411906: Pull complete 
    Digest: sha256:26c68657ccce2cb0a31b330cb0be2b5e108d467f641c62e13ab40cbec258c68d
    Status: Downloaded newer image for ubuntu:latest
    docker.io/library/ubuntu:latest

    rhyme@ip-10-199-122-109:~$ docker image ls
    REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
    ubuntu       latest    d2e4e1f51132   13 days ago   77.8MB

Running the image ubuntu without the *detach* flag opens bash:

    rhyme@ip-10-199-122-109:~$ docker run -it ubuntu
    root@180754b28733:/# ls
    bin   dev  home  lib32  libx32  mnt  proc  run   srv  tmp  var
    boot  etc  lib   lib64  media   opt  root  sbin  sys  usr
    root@180754b28733:/# exit
    exit
    rhyme@ip-10-199-122-109:~$ 

Otherwise, if the container is detached, it can be accessed later:

    rhyme@ip-10-199-122-109:~$ docker run -itd ubuntu
    e51bf16aed17f1fc03fe79108d1d811cd1a8e449798ae2f691df89e1aaa19f02
    rhyme@ip-10-199-122-109:~$ docker ps
    CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
    e51bf16aed17   ubuntu    "bash"    13 seconds ago   Up 12 seconds             hungry_greider

    rhyme@ip-10-199-122-109:~$ docker exec -it e5 bash
    root@e51bf16aed17:/# 

To exit, just type `exit` or `Ctrl+D`.

It can be executed command without the using the interactive mode:

    rhyme@ip-10-199-122-109:~$ docker exec e5 ls 
    bin
    boot
    dev
    etc
    ...

To inspect a container or an image:

    rhyme@ip-10-199-122-109:~$ docker inspect e5
    [
        {
            "Id": "e51bf16aed17f1fc03fe79108d1d811cd1a8e449798ae2f691df89e1aaa19f02",
            "Created": "2022-05-13T12:58:12.212417997Z",
            "Path": "bash",
            "Args": [],
            "State": {
                "Status": "running",
                "Running": true,
                "Paused": false,
    ...

    rhyme@ip-10-199-122-109:~$ docker images
    REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
    ubuntu       latest    d2e4e1f51132   13 days ago   77.8MB

    rhyme@ip-10-199-122-109:~$ docker image inspect d2
    [
        {
            "Id": "sha256:d2e4e1f511320dfb2d0baff2468fcf0526998b73fe10c8890b4684bb7ef8290f",
            "RepoTags": [
                "ubuntu:latest"
            ],
            "RepoDigests": [
                "ubuntu@sha256:26c68657ccce2cb0a31b330cb0be2b5e108d467f641c62e13ab40cbec258c68d"
            ],
            "Parent": "",
            "Comment": "",
            "Created": "2022-04-29T23:21:15.708209475Z",
    ... 

Deploying a web application like Jenkins:

    rhyme@ip-10-199-122-109:~$ docker pull jenkins/jenkins
    Using default tag: latest
    latest: Pulling from jenkins/jenkins
    6aefca2dc61d: Pull complete 
    4f7808c00553: Pull complete 
    ...

    rhyme@ip-10-199-122-109:~$ docker image ls
    REPOSITORY        TAG       IMAGE ID       CREATED       SIZE
    jenkins/jenkins   latest    f4079e4b5013   2 days ago    461MB
    ubuntu            latest    d2e4e1f51132   13 days ago   77.8MB

Check which are the ports exposed:

    rhyme@ip-10-199-122-109:~$ docker image inspect f4 | grep ExposedPorts -A 3
                "ExposedPorts": {
                    "50000/tcp": {},
                    "8080/tcp": {}
                },

Jenkins actually runs on the port 8080. This is the port of the Jenkins
container. To access the Jenkins container through the host machine, we need to
map it to an external port (i.e., the host port). The following command runs the
image `jenkins/jenkins` on detached mode, mapping the host port 80 to the
container port 8080:

    rhyme@ip-10-199-122-109:~$ docker run -d -p 80:8080 jenkins/jenkins
    4837e4f31df22702034bfb1cf4e70e0e8a3804bfd076f42551c0e8f4fe431850
    rhyme@ip-10-199-122-109:~$ docker ps
    CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                             NAMES
    4837e4f31df2   jenkins/jenkins   "/sbin/tini -- /usr/\u2026"   12 seconds ago   Up 11 seconds   50000/tcp, 0.0.0.0:80->8080/tcp   quizzical_pike
    e51bf16aed17   ubuntu            "bash"                   26 minutes ago   Up 26 minutes                                     hungry_greider

Now, if I open on the browser this URL <http://localhost:80/>, I\'ll get a Jenkins
page \"Getting started\". According to the instruction on this site, to access it,
one need to get the password in a given file. To do that:

    rhyme@ip-10-199-122-109:~$ docker exec 48 cat /var/jenkins_home/secrets/initialAdminPassword
    2f1135059bbc41ce937cc47742e745cc

## Docker networks

There are three ways to connect the container to the network:

* Bridge network: the default. Each container is behind the IP mask 172.17.0.0.
  Ex: `docker run ubuntu`
* None network: No network at all. Ex: `docker run ubuntu --network=none`
* Host network: The container gets the same network as the host. But you won\'t
  be able have multiple containers running on the same port like 8080! Ex: 
  `docker run ubuntu --network=host`

To list those networks:

    rhyme@ip-10-199-122-109:~$ docker network ls
    NETWORK ID     NAME      DRIVER    SCOPE
    ff58ae7bcd9c   bridge    bridge    local
    b70de08e4bed   host      host      local
    897199270ec8   none      null      local

    rhyme@ip-10-199-122-109:~$ docker network inspect ff
    [
        {
            "Name": "bridge",
            "Id": "ff58ae7bcd9cfc1d58f5e148fe53743359e4b34224234c3c83287c97232780a2",
            "Created": "2022-05-13T12:20:05.600109711Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": null,
                "Config": [
                    {
                        "Subnet": "172.17.0.0/16",
                        "Gateway": "172.17.0.1"
                    }
                ]
            },
    ...

When the docker engine is installed in the host machine, it will install an
interface called `docker0`:

``` {emphasize-lines="6"}
rhyme@ip-10-199-122-109:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 0a:e4:4c:99:27:1f brd ff:ff:ff:ff:ff:ff
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:74:8a:32:36 brd ff:ff:ff:ff:ff:ff
15: veth524366a@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default 
    link/ether 02:f6:0f:aa:a6:53 brd ff:ff:ff:ff:ff:ff link-netnsid 0
17: vethfd84047@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default 
    link/ether 06:17:90:5f:1f:69 brd ff:ff:ff:ff:ff:ff link-netnsid 1
rhyme@ip-10-199-122-109:~$ 
```

To check its IP:

    rhyme@ip-10-199-122-109:~$ ip addr
    ...
    3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
        link/ether 02:42:74:8a:32:36 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
           valid_lft forever preferred_lft forever
        inet6 fe80::42:74ff:fe8a:3236/64 scope link 
           valid_lft forever preferred_lft forever
    ...

In a container, there is a network namespace, which has its own network
interface which is conected through a virtual cable to the host network
interface `docker0`.

It is possible to create a new network isolated from the previous one. So,
containers linked to one network won\'t be able to access the containers linked
to the second network:

    rhyme@ip-10-199-122-109:~$ docker network create --driver=bridge --subnet=182.1.0.1/16 testingIsolatedNetwork
    e3a9af23f061daf37f22a9ea75697ba8d19c6d9d90a43647586d80203f57430d
    rhyme@ip-10-199-122-109:~$ docker network ls
    NETWORK ID     NAME                     DRIVER    SCOPE
    b19dbc4f1a7d   bridge                   bridge    local
    b70de08e4bed   host                     host      local
    897199270ec8   none                     null      local
    e3a9af23f061   testingIsolatedNetwork   bridge    local

    rhyme@ip-10-199-122-109:~$ docker network inspect e3
    [
        {
            "Name": "testingIsolatedNetwork",
            "Id": "e3a9af23f061daf37f22a9ea75697ba8d19c6d9d90a43647586d80203f57430d",
            "Created": "2022-05-13T16:50:46.851712035Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "182.1.0.1/16"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]

Creating a new container with the previously created network:

    rhyme@ip-10-199-122-109:~$ docker run -itd --name=testUbuntu --net=testingIsolatedNetwork ubuntu
    b47e9508376b07690cfa359a82eb1c3a28a3b9be313c6658407974df467fb719

    rhyme@ip-10-199-122-109:~$ docker ps
    CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
    b47e9508376b   ubuntu    "bash"    4 seconds ago   Up 3 seconds             testUbuntu
    e51bf16aed17   ubuntu    "bash"    4 hours ago     Up 5 minutes             hungry_greider

    rhyme@ip-10-199-122-109:~$ docker inspect b4
    [
        {
            ...
            "NetworkSettings": {
                "Bridge": "",
                ...
                "Networks": {
                    "testingIsolatedNetwork": {
                        "IPAMConfig": null,
                        "Links": null,
                        "Aliases": [
                            "b47e9508376b"
                        ],
                        "NetworkID": "e3a9af23f061daf37f22a9ea75697ba8d19c6d9d90a43647586d80203f57430d",
                        "EndpointID": "6d7ada31bd723e51da91a3f8b857bc2e93ef528e1af0358e09ee969b5787ab83",
                        "Gateway": "182.1.0.1",
                        "IPAddress": "182.1.0.2",
                        ...
                    }
                }
            }
        }
    ]

If the container is already running and we want to connect to a new network:

    rhyme@ip-10-199-122-109:~$ docker network connect e3 e5
    rhyme@ip-10-199-122-109:~$ docker inspect e5
    [
        {
            ...
            "NetworkSettings": {
                ...
                "Networks": {
                    "bridge": {
                        "Aliases": null,
                        "NetworkID": "b19dbc4f1a7d18a603cbc755ad6e87d600b73b8b2ae36cad4bc916e23fef59be",
                        "Gateway": "172.17.0.1",
                        "IPAddress": "172.17.0.2",
                        ...
                    },
                    "testingIsolatedNetwork": {
                        "Aliases": [
                            "e51bf16aed17"
                        ],
                        "NetworkID": "e3a9af23f061daf37f22a9ea75697ba8d19c6d9d90a43647586d80203f57430d",
                        "EndpointID": "4890c37503bfe111fe149e2e041846fb160be674530d4c34bd44d9247dee2e1c",
                        "Gateway": "182.1.0.1",
                        "IPAddress": "182.1.0.3",
                        ...
                    }
                }
            }
        }
    ]

Stoping and removing all the running containers:

    rhyme@ip-10-199-122-109:~$ docker stop $(docker ps -aq)
    b47e9508376b
    4837e4f31df2
    e51bf16aed17
    180754b28733

    rhyme@ip-10-199-122-109:~$ docker rm $(docker ps -aq)
    b47e9508376b
    4837e4f31df2
    e51bf16aed17
    180754b28733

Removing the created network:

    rhyme@ip-10-199-122-109:~$ docker network ls
    NETWORK ID     NAME                     DRIVER    SCOPE
    b19dbc4f1a7d   bridge                   bridge    local
    b70de08e4bed   host                     host      local
    897199270ec8   none                     null      local
    e3a9af23f061   testingIsolatedNetwork   bridge    local

    rhyme@ip-10-199-122-109:~$ docker network rm e3
    e3

To remove all created networks if they are not being used:

    rhyme@ip-10-199-122-109:~$ docker network prune
    WARNING! This will remove all custom networks not used by at least one container.
    Are you sure you want to continue? [y/N] y

    rhyme@ip-10-199-122-109:~$ docker network ls
    NETWORK ID     NAME      DRIVER    SCOPE
    b19dbc4f1a7d   bridge    bridge    local
    b70de08e4bed   host      host      local
    897199270ec8   none      null      local

Note that the default networks are preserved.

## Communicating between two docker containers

Creating a new bridge network and two new containers running CentOS. Initially,
one container will be using one network, and the second another network:

    rhyme@ip-10-199-122-109:~$ docker network create --driver=bridge --subnet=182.0.1.1/16 isolatedNetwork
    8fbbd32b9698366ec0c190bee3a62743cfa930483dc3dc170981aee369a016d5

    rhyme@ip-10-199-122-109:~$ docker run -itd centos
    Unable to find image 'centos:latest' locally
    latest: Pulling from library/centos
    a1d0c7532777: Pull complete 
    Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
    Status: Downloaded newer image for centos:latest
    3e3f352266f99945f35f779e8253186d66a69b0fe30cefd39ce33da7d5082d06

    rhyme@ip-10-199-122-109:~$ docker ps
    CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS         PORTS     NAMES
    3e3f352266f9   centos    "/bin/bash"   18 seconds ago   Up 8 seconds             interesting_johnson

    rhyme@ip-10-199-122-109:~$ docker run -itd --name=test1 --net=isolatedNetwork centos 
    df383244597344d0daf2f094ed74eb81bd3eeba25b7bd4eec9ca2e3ce3166549

    rhyme@ip-10-199-122-109:~$ docker ps
    CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS              PORTS     NAMES
    df3832445973   centos    "/bin/bash"   6 seconds ago   Up 5 seconds                  test1
    3e3f352266f9   centos    "/bin/bash"   2 minutes ago   Up About a minute             interesting_johnson

Because they are on different networks, the containers can\'t see each other:

    rhyme@ip-10-199-122-109:~$ docker exec -it 3e /bin/bash
    [root@3e3f352266f9 /]# ping test1
    ping: test1: Name or service not known
    [root@3e3f352266f9 /]# 

I can connect the first container to the network that is being used by the
second container, then they\'ll communicate:

    rhyme@ip-10-199-122-109:~$ docker network connect isolatedNetwork 3e

    rhyme@ip-10-199-122-109:~$ docker exec -it 3e /bin/bash
    [root@3e3f352266f9 /]# ping test1
    PING test1 (182.0.0.2) 56(84) bytes of data.
    64 bytes from test1.isolatedNetwork (182.0.0.2): icmp_seq=1 ttl=64 time=0.139 ms
    64 bytes from test1.isolatedNetwork (182.0.0.2): icmp_seq=2 ttl=64 time=0.076 ms
    64 bytes from test1.isolatedNetwork (182.0.0.2): icmp_seq=3 ttl=64 time=0.072 ms
    64 bytes from test1.isolatedNetwork (182.0.0.2): icmp_seq=4 ttl=64 time=0.065 ms

## Docker volumes

How to make docker persist the data? When a container is created, there are two
layers:

* Docker image layer (read-only)
* Container layer (writable)

When container is deleted, all the data is deleted as well. If I want to
preserve the data, we need to store it in a so called data volume. To do this,
we map the container layer to a volume. The volume resides on the docker host,
so it is external to the container. The created volumes are available in
`/var/lib/docker/volumes`. To create a volume:

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes$ docker volume create data_volume
    data_volume

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes$ sudo ls -ltr | grep data_volume
    drwx-----x 3 root root   4096 May 13 17:28 data_volume

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes$ cd data_volume
    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume$ sudo ls -ltr 
    total 4
    drwxr-xr-x 2 root root 4096 May 13 17:28 _data

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume$ cd _data
    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ sudo ls -ltr
    total 0

To create a container linked to the newly created volume, and map that volume to
the directory `/www`:

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ docker run -itd -v data_volume:/www ubuntu
    5a06ee073e7ee6a075bbca832c9009cbde6dc29f6f37a0e36f634fde4f287ecc

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ docker ps
    CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
    5a06ee073e7e   ubuntu    "bash"        4 seconds ago    Up 2 seconds              pedantic_shaw

Let's run this container and create a file in this volume:

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ docker exec -it 5a /bin/bash
    root@5a06ee073e7e:/# ls  
    bin   dev  home  lib32  libx32  mnt  proc  run   srv  tmp  var
    boot  etc  lib   lib64  media   opt  root  sbin  sys  usr  www
    root@5a06ee073e7e:/# cd www
    root@5a06ee073e7e:/www# echo "Test Data" > test.txt
    root@5a06ee073e7e:/www# ls
    test.txt
    root@5a06ee073e7e:/www# 
    exit

Note that this file is available in the volume folder, and it persists even
after stopping and removing the container where the file was created:

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ ls
    test.txt

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ docker stop 5a
    5a

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ ls
    test.txt

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ docker rm 5a
    5a

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ ls
    test.txt

It is possible to connect another container to that volume:

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ docker run -itd -v data_volume:/www ubuntu
    4e8027c6097cad79d5dd377a85ba3432ebbf78277e51770dcbb5d65f581a7e23

    rhyme@ip-10-199-122-109:/var/lib/docker/volumes/data_volume/_data$ docker exec -it 4e bash
    root@4e8027c6097c:/# cd www
    root@4e8027c6097c:/www# ls
    test.txt

This was the default volume mount. There is also the bind volume mount:

    rhyme@ip-10-199-122-109:~$ docker run -it -v /home/rhyme/data:/www ubuntu
    root@a0057d52bf0b:/# cd www
    root@a0057d52bf0b:/www# echo "Test data" > test.txt
    root@a0057d52bf0b:/www# ls
    test.txt
    root@a0057d52bf0b:/www# exit

    rhyme@ip-10-199-122-109:~$ ls data
    test.txt

No new volume is listed:

    rhyme@ip-10-199-122-109:~$ docker volume ls
    DRIVER    VOLUME NAME
    local     75d5ae2552776b37d6ae34e659941e7c01d91c9f9956777b43c460d05cb6f593
    local     40119234d3647a7aad0685353a657383c98e733dc3b9fa54785cae19bc40469e
    local     a1b3908774dd64b94ece53b3a7df11d039fb7cb07b7aff3a2eb2e59fa6d9f874
    local     c7c7f58dc3715d0809145f810cab2f079df20be6e2e77b349c88b1ca02913a67
    local     data_volume

Alternative syntax to create a container using a bind mount:

    rhyme@ip-10-199-122-109:~$ docker run -it --mount type=bind,source=/home/rhyme/data,target=/www ubuntu
    root@1cafb804c603:/# ls www
    test.txt
    root@1cafb804c603:/# 

To delete a volume:

    rhyme@ip-10-199-122-109:~$ docker volume rm data_volume
    data_volume
