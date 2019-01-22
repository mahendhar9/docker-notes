### Why use docker?
Docker makes it really easy to install and run software without worrying about setup or dependencies.

### What is docker?
Docker is a platform for creating and running containers.

### What is docker image?
A single file with all the dependencies and config required to run a program.
- It's a file system snapshot - a copy/paste of a specific set of directories or files.
- It also contains a `startup command` ( which invokes a process that has its own isolated set of hardware resources on host computer (which it got by namespacing and cgroups))

*`startup command` is executed inside the container after the container is created.

```
Docker image = (file system snapshot) + (startup command)
```

### Container 
A container is a program with its own isolated set of hardware resources.
- It is instance of an image.
- It runs a single program.
- It shares the host kernel with other containers.

The process (program) that is supposedly “inside” the container is actually running on the host in it’s own namespace, control group (cgroup), and union filesystem overlay. A cgroup limits a process to a specific set of resources.

* namespacing = isolating resources per process (or group of processes) ; like hard drive, users, etc
* cgroups = limit the amount of resources used per process ; like memory, cpu usage, etc.

--- 

### Docker client (docker cli)
Tool that we issue commands to. (in terminal)

### Docker server (docker daemon)
Tool that is responsible for creating images, running containers, etc.

---

``` 
docker run hello-world 
```
`hello-world` is the image

 1. The Docker client contacted the Docker daemon (server).
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.   
 3. The Docker daemon created a new container from that image which runs the executable (fs snapshot)  that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it to your terminal.

---

### Overriding default commands

```
docker run image-name command
```

*run = try to create and run a container
*image-name = name of the image to use for this container
*command = default command override

---

### List running containers
```
docker ps
```
### List all containers ever created
```
docker ps --all
```
---

### Container lifecycle

```
docker run = docker create + docker start
            ( create container from image + start the container )
```

- `docker create hello-world `
7d73e7be3251c2e06ce20652629c8e5d893e2d57ddaf464a03afc5ab69c41769 
(id of the container that was created) 
<br>
- `docker start -a id-of-container`
-a = watch for output from the container and print it out to the terminal

*By default,`docker run` shows all the logs or information coming out of the container, while `docker start` does not.

----
### Restarting stopped containers
1. Get the id of the container you want to restart by running `docker ps --all`
2. Then, `docker start -a id`
   
When you started a container and let it exit, you can start it back again, which will re-issue the default command that was used when the container was first created.

When you have a container that's already been created, you cannot replace the default command.
You cannot do `docker start -a 20652629c echo hi there`

---
### Removing all stopped containers

```
docker system prune
```

---
### Retrieving log outputs
```
docker logs container-id
```

`docker logs` is used to look at the (stopped) container and retrieve all the information that has been emitted from it.

`docker logs` does not re-run or restart the container, it just gets the record of all the logs that have been emitted from that container.


---

### Stopping containers
```
docker stop container-id
( gives the processes running inside the container a chance to shut down )
( if processes not shutting down, waits for 10 seconds and then issues the kill command )

docker kill container-id
( instantly stops the container )
```

---
### Execute an additional command in a running container

`docker run redis` starts redis server
- Now, if we want to execute another command inside that container,

```
docker exec -it container-id command

exec = run another command
-it = allows us to provide (enter/type) input to the container

eg: docker exec -it 20652629c redis-cli
```

Also, can be written as

```
docker exec -i -t container-id command

-i = when we execute this new command inside the container, we want to attach our terminal to the STDIN channel of that new running process (redis cli, here). Whatever we type in the terminal gets directed to the STDIN of redis cli and the information that comes out of STDOUT is output to the terminal.
-t = makes the text look pretty with indentation, etc.
```

---

### Getting a command prompt in a container
```
docker exec -it container-id sh
```
---

### Starting a container with a shell
```
docker run -it image-name sh
```

---
## Creating Docker images

### Docker file

- A Dockerfile is a text document that has all the configuration to define how a container should behave.

Docker file --> Docker client --> Docker server ==> Docker image


**Docker file template**
1. Specify a base image.
2. Run some commands to download and install additional programs.
3. Specify a command to run on container startup.

```dockerfile
FROM alpine
RUN apk add --update redis
CMD ["redis-server"]
```
---
### What is a Base Image?
- A base image is the image that is used to create all of our container images.
- Base image gives us an inital set of programs that we can then use to further customize our image. 
  (eg: like an operating system in a new computer. To install chrome, first we need an operating system.)
---

### Build Docker Image
`cd` into the folder that has the Docker file
```
docker build .

- The dot (.) is specifying the build context. 
- Build context is the set of files and folders that belong to our project.
- It's a set of files and folders that we want to encapsulate or wrap in this container.
```
---
### The Build Process
```
FROM alpine
- alpine image is downloaded from dockerhub (if it's not already in our local build cache)

RUN apk add --update redis
 - Takes the alpine image and creates a new temporary container out of it
 - `apk add --update redis` command is executed inside the new container as its primary running process
 - After redis is downloaded and installed, the current file system snapshot (which now contains redis) is taken and saved as a temporary image. The temporary container is now removed.
 - The output of everything inside this step is a new image that contains just the changes that we made during this step.

CMD ["redis-server"]
- Takes the temporary image generated in the previous step (that contains redis) and creates a new temporary container out of it.
- CMD is setting the primary command for the primary process of this container. The container does not execute `redis server`, it just sets this as the primary command.
- Takes a snapshot of this temporary container's file system and primary command and creates the final image before removing the temporary container.
```

- Every step along the way, with every additional instruction, we take the image that was generated in the previous step, we create a new container out of it, we execute a command in the container or make a change to its file system. We then take a snapshot of the container's file system + primary command to save it as an image for the next instruction along the chain. And when there are no more instructions to execute, the image that was generated during that last step is output from the entire process as the final image.
---