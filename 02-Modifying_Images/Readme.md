# Exercise 02 - Modifying Existing Images

Modify an existing Docker image, and commit it as a new one.

We'll modify the ubuntu:16.04 image to include the ping utility.

## Pull a base image locally

First download the `ubuntu:21.10` image with `docker pull`.
```
$ docker pull ubuntu:22.04
22.04: Pulling from library/ubuntu
a186900671ab: Pull complete 
Digest: sha256:58b87898e82351c6cf9cf5b9f3c20257bb9e2dcf33af051e12ce532d7f94e3fe
Status: Downloaded newer image for ubuntu:22.04
docker.io/library/ubuntu:22.04
```
## Modifying an image
Start the Ubuntu container with `bash`:
```
$ docker run -it ubuntu:22.04 /bin/bash
root@786b94c53c6d:/#
```
Try running `ping` from within the container:
```
root@dc80ca913947:/# ping www.google.com
bash: ping: command not found
```
It is not found as the Ubuntu image for Docker only has the bare minimum of software installed. 
Let's go ahead and install it with the `iputils-ping` package
```
# Run an update on the Ubuntu and install iputils-ping
$ apt-get update
# Install IP Utils
root@15cefc5f80f3:/# apt-get install iputils-ping
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libcap2-bin libpam-cap
The following NEW packages will be installed:
  iputils-ping libcap2-bin libpam-cap
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 76.8 kB of archives.
After this operation, 255 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
```
Hit `y` to continue the installation of `iputils-ping`

Once the installation completes, you can execute ping:
```
root@15cefc5f80f3:/# ping www.google.com
PING www.google.com (172.217.4.68) 56(84) bytes of data.
64 bytes from lga15s47-in-f68.1e100.net (172.217.4.68): icmp_seq=1 ttl=63 time=74.0 ms
64 bytes from lga15s47-in-f68.1e100.net (172.217.4.68): icmp_seq=2 ttl=63 time=104 ms
64 bytes from lga15s47-in-f68.1e100.net (172.217.4.68): icmp_seq=3 ttl=63 time=70.1 ms
^C
--- www.google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2009ms
rtt min/avg/max/mdev = 70.128/82.604/103.679/14.985 ms
```

## Committing your changes
What we did is not that complicated, and for now the changes only apply to the running container. 
Now, we want to have `ping` on all of our `ubuntu` containers, and not have to install `ping` all the time.
There are two ways to do this:
1. build a new image from scratch
2. commit a container state as a new image. 

In this section we're going to commit the container state as a new image. We'll look at new builds in the next section.

We already have our running container with `ping` installed. Find its ID with `docker ps` or `docker ps -a` if you stopped it.
```
docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS     NAMES
15cefc5f80f3   ubuntu:22.04   "/bin/bash"   13 minutes ago   Up 13 minutes             stoic_margulis
```
We'll be using `docker commit` command. `docker commit` takes a container, and allows you to commit its changes as a new image.
```
$ docker commit --help

Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image (default [])
      --help             Print usage
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
$
```
Pass the container ID, an author, commit message, and give it a name `<DockerHub username>/ubuntu-ping`:
```
$ docker commit -a 'George Paloulian' -m 'Added ping utility to Ubuntu 22.' 15c paloul/ubuntu-ping
sha256:d01961f3b4dd9b83a331b1d229e4801787afb95d0099de493cdede21e1e9c65d
```
Run `docker images` to see your new Docker image created and stored locally 
```
$ docker images
REPOSITORY                                TAG                                        IMAGE ID       CREATED         SIZE
paloul/ubuntu-ping                        latest                                     d01961f3b4dd   2 seconds ago   122MB
ubuntu                                    22.04                                      981912c48e9a   4 weeks ago     69.2MB
```
Run your new image and execute `ping` to verify the changes you've made

## Observations 
Take a look at the output from `docker images`. 
What do you notice between the original Ubuntu image and your customized image?

Building our own container images using Dockerfiles gives us flexibility to be more efficient.  