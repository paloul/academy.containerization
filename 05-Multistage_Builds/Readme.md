# Exercise 04 - Multistage Builds
With multi-stage builds, you use multiple FROM statements in your Dockerfile. Each FROM instruction can use a 
different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one 
stage to another, leaving behind everything you don't want in the final image.

In this exercise we are going to produce tiny image with nothing but the application binary inside for production use. 
None of the build tools required to build the application are included in the resulting image. Using containers to build
your applications will keep clutter off your development laptops as well as the CI/CD infrastructure when in production.

## Introduction 
Look at the following Dockerfile sample that contains two separate stages:
```
FROM golang:1.23
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```
The first stage builds the binary from Go source, and the second receives the binary from the first stage for execution.
There is no need for a separate build script, the build script is contained within the first stage, `go build -o /bin/hello ./main.go`

By default, the stages aren't named, and you can refer to them by their integer number determined by order of appearance, 
starting with 0 for the first FROM instruction. However, you can name your stages, by adding an AS <NAME> to the FROM instruction.
```
FROM golang:1.23 AS build
WORKDIR /src
COPY <<EOF /src/main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=build /bin/hello /bin/hello
CMD ["/bin/hello"]
```

The end result of the `docker build` command on this Dockerfile is one container image with the Go application ready to be run.
```
$ docker build -t go-hello -f Dockerfile0 .
# Wait for the container to build, then check your images

$ docker images
REPOSITORY                                TAG                                        IMAGE ID       CREATED         SIZE
go-hello                                  latest                                     2a82672a2858   6 seconds ago   2.15MB
```
Start the new container and verify the Go binary runs successfully.

## Hands-On Activity
In this section we're going to introduce a more involved build process using a Scala/JVM application, showing off how
using multi-stage builds can drastically reduce container sizes on more involved applications.

First let's look at the `Dockerfile1` definition:
```
FROM sbtscala/scala-sbt:eclipse-temurin-alpine-21.0.2_13_1.10.2_3.5.1
LABEL authors="George Paloulian"

# Set the working directory inside the container
WORKDIR /grpc-streaming-mod

# Get the source code directly into current working directory /grpc-streaming-mod
RUN git clone https://github.com/paloul/grpc.streaming.mod.git .

# Use SBT to compile
RUN sbt clean compile

# Expose the port
EXPOSE 8080

ENTRYPOINT ["sbt", "runMain edu.caltech.cast.indy.GreeterServer"]
```
A few new things in this Dockerfile:
1. Using a new base image `sbtscala/scala-sbt` that comes with Java and other dev tools by default. 
2. Creating a Working Directory with `WORKDIR` 
3. Cloning a public repo directly into the working directory
4. Compiling the source using [SBT](https://www.scala-sbt.org/) 
5. Exposing the port 8080 for the application to be able to receive TCP traffic
6. Defining the Entrypoint for the container as a `sbt runMain` command

Build the container and check the container image. 
How large is the container image size? Do you think we can make that better? How?
```
$ docker build -t grpc-streaming-mod-1 -f Dockerfile1 .

$ docker images

# You can start the image with the `run` command. 
# Map your local port to the container port with '-p' 
# to allow the container to receive traffic
$ docker run -p 8080:8080 grpc-streaming-mod-1
```

### Create your own multi-stage build
In the SBT based Dockerfile above, we used SBT to run the application `sbt "runMain edu.caltech.cast.indy.GreeterServer"`. 

At this point, you should have all the background to create a multi-stage build by yourself. The project at
[grpc.streaming.mod](https://github.com/paloul/grpc.streaming.mod.git) uses [SBT Native Packager](https://www.scala-sbt.org/sbt-native-packager/)
to generate JARs and helper run scripts. You can explore its functionality with interactive containers and `bash`.  

The goal of this exercise is to create a multi-stage build where the final container only has the necessary JAR files 
required to run the application. Expand on `Dockerfile1` and create your own multi-stage build with `Dockerfile2`.

Build your own image and check the image size. Report on the image size. 
```
$ docker build -t grpc-streaming-mod-2 -f Dockerfile2 .

$ docker images

# Run the new image to see if it worked
$ docker run -p 8080:8080 grpc-streaming-mod-2
```

*Hint*: The final image can base itself from a different base image as long as Java is available, i.e. 
[eclipse-temurin](https://hub.docker.com/_/eclipse-temurin). Try and search for the right tag you want to use. 
Look for OS version and the differences between JDK/JRE. 