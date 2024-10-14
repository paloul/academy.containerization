# Exercise 01 - Running Containers

Basics of pulling images, starting, stopping, and removing containers

## Working with images
Before running containers, we'll need to pull some images and have them locally on our machine.

Let's see what images we have currently on our machine, run `docker images`:

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

On a fresh Docker installation, you should have no images, or you should see at least
the `hello-world` Docker image from the previous step.  

We can search for images using `docker search <keyword>`:
```
$ docker search ubuntu
NAME                             DESCRIPTION                                     STARS     OFFICIAL
ubuntu                           Ubuntu is a Debian-based Linux operating sys…   17319     [OK]
ubuntu/squid                     Squid is a caching proxy for the Web. Long-t…   99        
ubuntu/nginx                     Nginx, a high-performance reverse proxy & we…   119       
ubuntu/cortex                    Cortex provides storage for Prometheus. Long…   4         
ubuntu/zookeeper                 ZooKeeper maintains configuration informatio…   13        
ubuntu/kafka                     Apache Kafka, a distributed event streaming …   51        
ubuntu/apache2                   Apache, a secure & extensible open-source HT…   78        
ubuntu/bind9                     BIND 9 is a very flexible, full-featured DNS…   96        
```

Run `docker pull ubuntu:16.04` to pull an image of Ubuntu 16.04 from DockerHub.
```
$ docker pull ubuntu:16.04
16.04: Pulling from library/ubuntu
828b35a09f0b: Pull complete 
66927c6d1d3d: Pull complete 
000560be9165: Pull complete 
6225a0253717: Pull complete 
Digest: sha256:1f1a2d56de1d604801a9671f301190704c25d604a416f59e03c04f5c6ffee0d6
Status: Downloaded newer image for ubuntu:16.04
docker.io/library/ubuntu:16.04

What's Next?
  1. Sign in to your Docker account → docker login
  2. View a summary of image vulnerabilities and recommendations → docker scout quickview ubuntu:16.04
```
We can also pull another version of Ubuntu by specifiying a different tag `docker pull ubuntu:16.10`
```
docker pull ubuntu:16.10
16.10: Pulling from library/ubuntu
dca7be20e546: Pull complete 
40bca54f5968: Pull complete 
61464f23390e: Pull complete 
d99f0bcd5dc8: Pull complete 
120db6f90955: Pull complete 
Digest: sha256:8dc9652808dc091400d7d5983949043a9f9c7132b15c14814275d25f94bca18a
Status: Downloaded newer image for ubuntu:16.10
docker.io/library/ubuntu:16.10

What's Next?
  1. Sign in to your Docker account → docker login
  2. View a summary of image vulnerabilities and recommendations → docker scout quickview ubuntu:16.10
```

Eventually, your machine can collect too many Docker images. Delete them with 
`docker rmi <IMAGE_ID>`
```
$ docker rmi 7d3f705d307c
Untagged: ubuntu:16.10
Untagged: ubuntu@sha256:8dc9652808dc091400d7d5983949043a9f9c7132b15c14814275d25f94bca18a
Deleted: sha256:7d3f705d307c7c225398e04d4c4f8512f64eb8a65959a1fb4514dfde18a047e7
Deleted: sha256:d9db289b9342d9617596cd6ee3bba988629e24d9afa5db4e4b0e4e491c65007d
Deleted: sha256:a87725e8597b97f2399bc3aa50b0e2eec903b8ce19055668d3befb012918205c
Deleted: sha256:38cf10a2801529348366953e9b933d3524360dedc91d3e4d5d7f941da0c973c9
Deleted: sha256:61172966249d43026dbd017eec3a9575e37bddf8a269a9f09ecb559d7bfe7fef
Deleted: sha256:57145c01eb80040fdd0a24cde20af4788605b49593188d4f7efab099af89a08e
```
You can delete images by tag or by a partial image ID. In the previous example, the following would have been equivalent:
 - `docker rmi 7d`
 - `docker rmi ubuntu:16.10`

A nice shortcut I like to use to clean up all images `docker rmi $(docker images -a -q)`

## Running containers
Using the Ubuntu 16.04 image we pulled, we can run our first container. 
Unlike a traditional virtualization software like VirtualBox or VMWare, containers need an `ENTRYPOINT` or simply a 
command to run, either long-running or short-lived. The container will automatically "stop", once the command terminates.

The entrypoint command can be anything, as long as it exists on the image: `echo 'Hello world!'`
```
$ docker run ubuntu:16.10 /bin/echo 'Hello world!'
Hello world!
```
Let's do something a bit more interactive. Run `docker run ubuntu:16.10 /bin/bash`
```
$ docker run ubuntu:16.10 /bin/bash
```
What do you notice? `bash` is an interactive program. This is default behavior for the `run` command, it is not interactive.
Since interactivity is not enabled, the `bash` application notices this and quits, and then the container stops. 

Add the `-it` flag to the `run` command to enable interactivity. 
```
$ docker run -it ubuntu:16.10 /bin/bash
root@b5662f7d7838:/#
```
Execute `docker ps` to see the container running with `bash`
```
$ docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS     NAMES
b5662f7d7838   ubuntu:16.10   "/bin/bash"   11 minutes ago   Up 11 minutes             silly_hugle
```
Enter `exit` to terminate the `bash` application. Execute the `docker ps` command again.

What if we want to have the container running indefinitely, potentially to debug a running application. 

You can run an interactive container as detached. Detached mode, shown by the option --detach or -d, 
means that a Docker container runs in the background of your terminal. It does not receive input or display output.
```
$ docker run -itd ubuntu:16.10 /bin/sleep 3600
e53f81943d04db2118e4b08a748bf6c9d84bc9af4f2d260d21e258ecb6c9facd
```
Now that the container is running in the background, what if we want to reattach to it?

Conceivably, if this were something like a web server or other process where we might like to inspect logs while it runs, 
it'd be useful to run something on the container without interrupting the current process. 

To this end, there is another command, called `docker exec`. `docker exec` runs a command within a container that is 
already running. It works exactly like `docker run`, except instead of taking an image ID, it takes a container ID.
```
$ docker exec -it e53 /bin/bash
root@e53f81943d04:/# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0 310852  4192 pts/0    Ss+  18:42   0:00 /run/rosetta/rosetta /bin/sleep /bin/sleep 3600
root         7  0.7  0.0 325160  6516 pts/1    Ss   18:43   0:00 /run/rosetta/rosetta /bin/bash /bin/bash
root        17  0.0  0.0 341332  6116 pts/1    R+   18:43   0:00 /bin/ps ps aux
```

## Stopping and Deleting containers
You can stop running containers with `docker stop`
```
$ docker stop -f e53
```
The `-f` argument will help terminate processes that might be stuck, like the `sleep` command in our previous case.

A container needs to be stopped to be able to remove it. Use `docker rm` to remove a container from your local machine.

You can see all (running or stopped) containers with `docker ps -a`
```
$ docker ps -a
....
$ docker rm e53
e53
$ docker ps -a
....
```
For convenience, a shortcut to delete all stopped containers on your machine:
```
docker rm $(docker ps -a)
```