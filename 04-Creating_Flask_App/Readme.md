# Exercise 04 - Building and Hosting a Flask Web Application
In this exercise we are going to dockerize a minimalist flask application. We will see how to expose ports and map
them to your localhost during runtime in order to interact with the application.

[Flask](https://pypi.org/project/Flask/) is a lightweight WSGI web application framework. It is designed to make 
getting started quick and easy, with the ability to scale up to complex applications.

## The Flask application
Take a look at the `flask_app.py`
```
from flask import Flask, jsonify

app = Flask(__name__)


@app.route("/hello", methods=["GET"])
def say_hello():
    return jsonify({"msg": "Hello from Flask"})


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```
The application sets up one route, `/hello`, accessible via a GET method, and returns back a JSON message. As you can see, 
the application gets started on port `5000`. We will expose this port in the Dockerfile in order to make the service hosted
by the Flask application reachable from the outside. 

## The Dockerfile
Take a look at the `Dockerfile`
```
FROM python:3.9-slim-buster

# Set the working directory
WORKDIR /app

# Copy our necessary files for the application
COPY ./flask_app.py /app
COPY ./requirements.txt /app

# Install our dependencies
RUN pip install -r requirements.txt

# Expose the 5000 port on the container.
EXPOSE 5000

ENV FLASK_APP=flask_app.py

CMD ["flask", "run", "--host", "0.0.0.0"]
```
There are a couple of new additions to this Dockerfile. 
1. WORKDIR  
This is a helper function to help minimize repeated paths as it sets the working directory for any 
RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile. 
2. COPY  
It copies new files or directories from source to the filesystem of the image 
3. EXPOSE  
It informs Docker that the container listens on the specified network ports at runtime. Default is TCP.
4. ENV  
Set an environment variable for use inside the container during runtime

## Build the image from the Dockerfile
```
$ docker build -t flask-app -f Dockerfile .
[+] Building 7.1s (10/10) FINISHED                                                                                     docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                   0.0s
 => => transferring dockerfile: 436B                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim-buster                                                              2.3s
 => [internal] load .dockerignore                                                                                                      0.0s
 => => transferring context: 2B                                                                                                        0.0s
 => [1/5] FROM docker.io/library/python:3.9-slim-buster@sha256:320a7a4250aba4249f458872adecf92eea88dc6abd2d76dc5c0f01cac9b53990        2.5s
 => => resolve docker.io/library/python:3.9-slim-buster@sha256:320a7a4250aba4249f458872adecf92eea88dc6abd2d76dc5c0f01cac9b53990        0.0s
 => => sha256:14aea17807c4c653827ca820a0526d96552bda685bf29293e8be90d1b05662f6 2.65MB / 2.65MB                                         0.7s
 => => sha256:03b001ef06cfba07019144f7f4db343a9002007cff20d4a6ae6dc0fb095742f1 11.13MB / 11.13MB                                       1.4s
 => => sha256:320a7a4250aba4249f458872adecf92eea88dc6abd2d76dc5c0f01cac9b53990 988B / 988B                                             0.0s
 => => sha256:dff25f8c8be28a31c6aa1deb28949e41af619352ed9c98d96f20059cc2b963ab 1.37kB / 1.37kB                                         0.0s
 => => sha256:bb1bad240fdfabd5834a73653c10a0696a0b652b81b7e80196400143f86a9649 6.83kB / 6.83kB                                         0.0s
 => => sha256:d191be7a3c9fa95847a482db8211b6f85b45096c7817fdad4d7661ee7ff1a421 25.92MB / 25.92MB                                       1.2s
 => => sha256:361a81f618250e198664f2dae40e4ed6794adfd9785dad752d3ab134a7017aac 242B / 242B                                             1.0s
 => => sha256:2eaed1ae01443ee6fcb6800a70c65d65fa2d3755e8deedd6958759ff27b8b22b 3.14MB / 3.14MB                                         1.5s
 => => extracting sha256:d191be7a3c9fa95847a482db8211b6f85b45096c7817fdad4d7661ee7ff1a421                                              0.7s
 => => extracting sha256:14aea17807c4c653827ca820a0526d96552bda685bf29293e8be90d1b05662f6                                              0.1s
 => => extracting sha256:03b001ef06cfba07019144f7f4db343a9002007cff20d4a6ae6dc0fb095742f1                                              0.2s
 => => extracting sha256:361a81f618250e198664f2dae40e4ed6794adfd9785dad752d3ab134a7017aac                                              0.0s
 => => extracting sha256:2eaed1ae01443ee6fcb6800a70c65d65fa2d3755e8deedd6958759ff27b8b22b                                              0.1s
 => [internal] load build context                                                                                                      0.0s
 => => transferring context: 412B                                                                                                      0.0s
 => [2/5] WORKDIR /app                                                                                                                 0.2s
 => [3/5] COPY ./flask_app.py /app                                                                                                     0.0s
 => [4/5] COPY ./requirements.txt /app                                                                                                 0.0s
 => [5/5] RUN pip install -r requirements.txt                                                                                          2.0s
 => exporting to image                                                                                                                 0.1s 
 => => exporting layers                                                                                                                0.1s 
 => => writing image sha256:aa1629436651822f5a7f53fe6cffde4a36a72810d21cb0e7baf06d54d5aeb6a0                                           0.0s 
 => => naming to docker.io/library/flask-app
```

## Run the Flask container
With the image built from the Dockerfile you can run a new container. In order to communicate with our application running
inside the container we have to map a port from our host machine (laptop) to the container. You can use the `-p` or `--publish` 
flag to specify port mapping to the `docker run` command. The syntax is as follows: `-p [host_port]:[container_port]`
```
$ docker run -p 5000:5000 flask-app
 * Serving Flask app 'flask_app.py'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.17.0.2:5000
Press CTRL+C to quit
```
Open your browser and visit http://localhost:5050/hello to see the Flask application working within the container.