# Docker Cheat Sheet

## Installation

1. Download and install Docker Desktop from [Docker Desktop Download](https://www.docker.com/products/docker-desktop/)

## Docker Hub

1. Create a Docker Hub account at [Docker Hub](https://docs.docker.com/engine/reference/commandline/login/)
2. Login to Docker Hub:
    ```bash
    docker login
    ```

## Building Images

Build an image from a Dockerfile:
```bash
docker build -t <image_name> .

Example:

bash
Copy code
docker build -t pradeeps717/hey-nodejs:0.0.1.RELEASE .
Running Containers
List running containers:

bash
Copy code
docker container ls
Example output:

bash
Copy code
CONTAINER ID    IMAGE            COMMAND             CREATED          STATUS              PORTS             NAMES
bebef9c26cf8d    pradeeps717/hey-nodejs:0.0.1.RELEASE   "docker-entrypoint.s…"    16 seconds ago    Up 15 seconds      0.0.0.0:3000->3000/tcp    naughty_goldberg
Run a container from an image:

bash
Copy code
docker container run -d -p <host_port>:<container_port> <image_name>
Example:

bash
Copy code
docker container run -d -p 3000:3000 pradeeps717/hey-nodejs:0.0.1.RELEASE
Stop a container:

bash
Copy code
docker container stop <container_id>
Example:

bash
Copy code
docker container stop bebef9c26cf8
Pushing Images to Docker Hub
Push an image to Docker Hub:

bash
Copy code
docker push <username>/<image_name>
Example:

bash
Copy code
docker push pradeeps717/hey-nodejs:0.0.1.RELEASE
Dockerfile Example
dockerfile
Copy code
FROM node:latest

WORKDIR /app

COPY package*.json /app

RUN npm install

COPY . /app

EXPOSE 3000

CMD ["npm", "start"] // start the server (CMD node app.js)
Command Summary
Command	Purpose
docker build	Builds an image from a Dockerfile
docker container ls	Lists running containers
docker container run	Runs a container from an image
docker container stop	Stops a container
docker push	Pushes an image to Docker Hub