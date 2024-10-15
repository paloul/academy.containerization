# Exercise 03 - Building New Images

Similar to the Modifying Images section, we are going to add the `ping` utility to the `ubuntu` image, but starting from
scratch with `Dockerfile`s instead of modifying a running container.

The Dockerfile is a "recipe" of instructions, that contains a list of commands describing how to build a new image.

## Pull a base image locally
First download the `ubuntu:22.04` image with `docker pull`.
```
$ docker pull ubuntu:22.04
22.04: Pulling from library/ubuntu
a186900671ab: Pull complete 
Digest: sha256:58b87898e82351c6cf9cf5b9f3c20257bb9e2dcf33af051e12ce532d7f94e3fe
Status: Downloaded newer image for ubuntu:22.04
docker.io/library/ubuntu:22.04
```
## Clean up any previous running containers and images
```
# Stop all running containers
$ docker stop $(docker ps -a)

# Remove all existing containers
$ docker rm $(docker ps -a)

# Remove images 
$ docker rmi $(docker images -a -q)
```

## Create the Dockerfile
Create a new empty file named `Dockerfile`
```
touch Dockerfile
# Or you can use any text IDE of your choosing
```
In the Dockerfile add the following:
```
FROM ubuntu:22.04
LABEL author="George Paloulian"
```
The FROM directive specifies what base image this new image will be built upon. (Ubuntu in our case.)

The LABEL directive adds a label to the image, useful for adding arbitrary metadata to the image.

Now, we can add `RUN` commands that alter the layers of the image. If you recall, container images are layers built on 
top of each other. Each of these layers, once created, are immutable. And each layer in an image contains a set of 
filesystem changes - additions, deletions, or modifications. In our case here, the `RUN` commands create layers within 
the image.

The `RUN` directive runs a command inside the image, and rolls any changes to the filesystem into a commit as a layer. 
A typical Dockerfile will contain several `RUN` statements, each committing their changes on top of the previous.

To install `ping`, we'll need to run `apt-get update` and `apt-get install`. Add the following to the Dockerfile:
```
RUN apt-get update

RUN apt-get install -y iputils-ping
```
The `-y` flag will instruct `apt-get` to run non-interactively, prompting `y` to any potential questions such as 
"Are you Sure?" during installation of any packages.

The Dockerfile should now look like this:
```
FROM ubuntu:22.04
LABEL author="George Paloulian"

RUN apt-get update

RUN apt-get install -y iputils-ping
```

## Building the Image from Dockerfile
We use the `docker build` command to build images. It reads in a Dockerfile and executes the commands for the image.
```
# In the folder where your Dockerfile can be found. Feel free to change the username `paloul` in paloul/ubuntu-ping
$ docker build -t 'paloul/ubuntu-ping' .
[+] Building 14.0s (7/7) FINISHED                                                                                                                                               docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                                                                            0.0s
 => => transferring dockerfile: 180B                                                                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/ubuntu:22.04                                                                                                                                 1.6s
 => [internal] load .dockerignore                                                                                                                                                               0.0s
 => => transferring context: 2B                                                                                                                                                                 0.0s
 => [1/3] FROM docker.io/library/ubuntu:22.04@sha256:58b87898e82351c6cf9cf5b9f3c20257bb9e2dcf33af051e12ce532d7f94e3fe                                                                           1.8s
 => => resolve docker.io/library/ubuntu:22.04@sha256:58b87898e82351c6cf9cf5b9f3c20257bb9e2dcf33af051e12ce532d7f94e3fe                                                                           0.0s
 => => sha256:a186900671ab62e1dea364788f4e84c156e1825939914cfb5a6770be2b58b4da 27.36MB / 27.36MB                                                                                                1.1s
 => => sha256:58b87898e82351c6cf9cf5b9f3c20257bb9e2dcf33af051e12ce532d7f94e3fe 1.34kB / 1.34kB                                                                                                  0.0s
 => => sha256:7c75ab2b0567edbb9d4834a2c51e462ebd709740d1f2c40bcd23c56e974fe2a8 424B / 424B                                                                                                      0.0s
 => => sha256:981912c48e9a89e903c89b228be977e23eeba83d42e2c8e0593a781a2b251cba 2.31kB / 2.31kB                                                                                                  0.0s
 => => extracting sha256:a186900671ab62e1dea364788f4e84c156e1825939914cfb5a6770be2b58b4da                                                                                                       0.6s
 => [2/3] RUN apt-get update                                                                                                                                                                    8.5s
 => [3/3] RUN apt-get install -y iputils-ping                                                                                                                                                   1.9s
 => exporting to image                                                                                                                                                                          0.1s
 => => exporting layers                                                                                                                                                                         0.1s 
 => => writing image sha256:58af6660875299ac39378ee99347a65a7f2fa1c2bcd0cf213f5483be0455be6d                                                                                                    0.0s 
 => => naming to docker.io/paloul/ping 
```
The `.` at the end of command instructs `docker build` to look for the Dockerfile in the current directory we're in. 

Docker layers each commit on top of the other. By doing so, it can keep image sizes small, and when 
rebuilding images, it can even reuse commits that are unaffected by changes to make builds run quicker. 
Run the build command again to see `docker build` leverage cached layers.
```
$ docker build -t 'paloul/ubuntu-ping' .
[+] Building 0.4s (7/7) FINISHED                                                                                                                                                docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                                                                            0.0s
 => => transferring dockerfile: 180B                                                                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/ubuntu:22.04                                                                                                                                 0.4s
 => [internal] load .dockerignore                                                                                                                                                               0.0s
 => => transferring context: 2B                                                                                                                                                                 0.0s
 => [1/3] FROM docker.io/library/ubuntu:22.04@sha256:58b87898e82351c6cf9cf5b9f3c20257bb9e2dcf33af051e12ce532d7f94e3fe                                                                           0.0s
 => CACHED [2/3] RUN apt-get update                                                                                                                                                             0.0s
 => CACHED [3/3] RUN apt-get install -y iputils-ping                                                                                                                                            0.0s
 => exporting to image                                                                                                                                                                          0.0s
 => => exporting layers                                                                                                                                                                         0.0s
 => => writing image sha256:58af6660875299ac39378ee99347a65a7f2fa1c2bcd0cf213f5483be0455be6d                                                                                                    0.0s
 => => naming to docker.io/paloul/ubuntu-ping
```

## Optimizing Image Sizes
Execute `docker images` to see your newly created image:
```
$ docker images
REPOSITORY                                TAG                                        IMAGE ID       CREATED         SIZE
paloul/ubuntu-ping                        latest                                     58af66608752   5 minutes ago   122MB
ubuntu                                    22.04                                      981912c48e9a   4 weeks ago     69.2MB
```
The image size almost doubled... from 69MB to 122MB. The reason goes back to the idea of the layers themselves as `RUN` 
commands commit all file system changes, that includes temporary data, logs, etc, which might be completely unnecessary 
for our final image.  

Let's add a change to the Dockerfile. Add a new `RUN` to clean up `apt-get` fluff files we don't need. 
```
FROM ubuntu:22.04
LABEL author="George Paloulian"

RUN apt-get update

RUN apt-get install -y iputils-ping

RUN apt-get clean \
    && cd /var/lib/apt/lists && rm -fr *Release* *Sources* *Packages* \
    && truncate -s 0 /var/log/*log
```
Save the above as `Dockerfile2` or any name you like and use it to create a new image by running `build` again. Use the 
`-f` argument to specify a different Dockerfile, i.e. `Dockerfile2`
```
$ docker build -t 'paloul/ubuntu-ping2' -f Dockerfile2 .
```
View the images again with `docker images`:
```
$ docker images
REPOSITORY                                TAG                                        IMAGE ID       CREATED          SIZE
paloul/ubuntu-ping2                       latest                                     b2ce7f7d2a8c   4 seconds ago    122MB
paloul/ubuntu-ping                        latest                                     58af66608752   17 minutes ago   122MB
ubuntu                                    22.04                                      981912c48e9a   4 weeks ago      69.2MB
```
What? The images are the same size!

Because the filesystem commits are layered one after another, if there is spare data from a previous commit, 
it doesn't matter if you delete it in a subsequent `RUN` directive. It is a permanent part of the image history, 
and thus the bloated image size.

We have to collapse the related `RUN` directives for `apt-get` all together as one.
```
$ docker build -t 'paloul/ubuntu-ping3' -f Dockerfile3 .
```
View the images again with `docker images` and report what you see. 

## Why Optimize Size?
1. Faster deployments  
   Smaller images are quicker to download, transfer, and load into the container runtime, which speeds up the deployment process. 
2. Improved build times  
   Efficient Dockerfiles can reduce build times, especially with much more complicated build pipelines, where layers are built on top of other layers.
3. Better storage utilization  
   Larger images take up more storage and storage equals $$$. While storage is cheap especially in the cloud, modern CI/CD pipelines can generate multiple builds in short amounts of time.
4. Reduced network bandwidth  
   Larger images require more network bandwidth to transfer, adding to costs. 
5. Improved security  
   Smaller images typically contain fewer components and fewer potential security vulnerabilities. Removing unwanted libraries can mitigate risk.
6. Improved application performance  
   Optimized containers use system resources more effectively.
