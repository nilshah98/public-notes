---
date: 2024-02-01
tags:
  - computer-science
  - blog
  - to-do
---

# Context

- You run a business, that needs to host several of your applications *(software)* online.
- While running *(executing)* these application, they might have some secret files, or API Keys, etc...
- What are the various options you have, and how do they match across various criteria like →
    1. cost → cheapest way to run your apps ?
    2. maintenance → how easy is to manage your app instances ?
    3. security → If you are sharing a machine with other business' applications, how do you ensure your files are kept secret ?
- Other misc points, include what happens if in shared hosting, other business' apps hog up your resources, or they are on the same network ? ...
- Taking all of these points into account, what can be a feasible solution to host your app ?

# Why Containers ?

## Notes

### Bare metal

- You access the whole hardware and software.
- Pros ⇒
    1. Your apps interact directly with the bare-metal, hence they have very low latency
    2. Can consume all the resources of the bare-metal
- Cons ⇒
    1. Need a whole team to manage your own server, updates, configurations, etc ...
    2. If you need more computing power, need to setup a whole server and attach to it, which consumes a lot of time and energy
    3. At times you might not consume the whole computing power but are still paying the bills for it
    4. If you share the bare-metal with other services, they are not sandboxed and can access your files and resources. So, if their service breaks it can bring the whole bare-metal and your service down as well
- Ex : Day to day personal computer hosting

### VM

- You spin multiple guest OS's which use hypervisor to interact with the bare-metal. You manage both the software (VM's) as well as the bare-metal
- Pros ⇒
    1. You can assign different services different OS, and hence their own different resources.
    2. This provides sandboxing as well, and optimal use of resources between them.
- Cons ⇒
    1. Need a whole team to manage your own server, updates, configurations, etc ...
    2. If you need more computing power, need to setup a whole server and attach to it, which consumes a lot of time and energy
    3. You do not interact directly with the bare-metal, instead are interacting with hypervisor *(abstraction)*, which in turn interacts with the bare-metal. This adds a little latency to the services.
- Ex : Spinning VM's on your own personal machine

### Public Cloud

- You spin multiple guest OS's which use hypervisor to interact with the bare-metal. You manage the software (VM's) and the cloud services manage the hardware.
- Pros ⇒
    1. You can assign different services different OS, and hence their own different resources.
    2. This provides sandboxing as well, and optimal use of resources between them.
- Cons ⇒
    1. You do not interact directly with the bare-metal, instead are interacting with hypervisor *(abstraction)*, which in turn interacts with the bare-metal. This adds a little latency to the services.
    2. Porting a VM to a new machine can be tough, as you need to clone the whole VM state and copy them, which can be in GB's
- Ex : AWS, Azure, GCP, etc ...

### Containers

- Instead of running guest OS which interact with hypervisor, you use unix utilities to provide sandboxing *(container engine)* and each service only has the essential binaries needed by them *(container)*
- Pros ⇒
    1. You can assign different services different containers.
    2. This provides sandboxing as well, and optimal use of hardware resources between them.
    3. Since containers only contain the most essential needed things, one can easily create an image in MB's and can easily port and move those around.
- Cons ⇒
    1. You do not interact directly with the bare-metal or hypervisor, you are interacting with a container engine, which is much more light-weight than running whole OS and interacting with hypervisor.

## Questions

- What are host OS ?
    - Host operating system is the main operating system that resides in a computer and interacts directly with the hardware (mare-metal)
    - Hence, they are more performant and have access to greater resources
- What are guest OS ?
    - Guest OS are OS's that you run in a virtualized environment.
    - They do not interact directly with the hardware, and rather interact with the hypervisor, which then interact with the hardware
    - They are complete replicas of host OS, just sitting on top of Hypervisor.
    - Also, there are two types of hypervisor depends on who they interact with →
        1. Type 1 : bare metal
        2. Type 2 : host OS
    - Since guest OS is not interacting directly with bare-metal there is added latency, as well as resource can be distributed if multiple guest OS are executing
- VM vs Container ?
    - In VM you spawn up whole new OS *(guest OS),* and these guest OS provide sandboxing *(isolation of files, networks, processes, etc ... )*
    - In Containers, you use unix utilities to provide sandboxing *(isolation)* →
        1. chroot
        2. namespace
        3. cgroup
    - Since the Container Engine *(above utilities)* are used to provide isolation, only the most essential files are copied over to the container, hence these are lot smaller than VM's and hence faster.
    
    ![Intro%20to%20Containers%20a43bef07138445878431add2510b56ff/Untitled.png](Intro%20to%20Containers%20a43bef07138445878431add2510b56ff/Untitled.png)
    

## Summary

<aside>
💡 Containers use chroot, namespace and cgroups, to provide sandboxing instead of running guest OS's. Hence, they are lightweight *(in MB's and spin up in minutes)*, flexible and easy to move around.

</aside>

---

# Unix Utilities for Sanboxing

## Notes

### chroot

- Used to isolate your file-system, ie. set a root directory for a new process. This way the new process cannot see out side of that root directory, and hence cannot see what else lies on the host machine.
- When you spin up a new root, it's completely empty and you won't even have binaries to execute commands like `ls` and `cat` , you will need to manually copy those binaries and shared libraries in `/bin` and `/lib`  directories.
- Even `bash` won't work, and you will need to copy it's binaries and shared libs in the appropriate folders of the new root.
- This isolates the file-system. And hence is often called `chroot-jail` and is often termed as `jailing` a process.

### namespace

- Namespaces helps with various resource isolation.
- Chroot only isolated the file-system, while they still have access to view all the processes, kill processes, unmount filesystem and potentially hijack processes.
- Hence, for resource isolation you use namespace, which can isolate processes, networks, mounts, users. This helps to resolve the other issues surrounding process and network isolation.

### cgroups

- cgroups is a mechanism to control system resources.
- Difference between namespace and cgroups is, namespace gives you permissions to those resources, so they are either 0 or 1. So, if you have access to RAM, you can access the whole RAM via namespaces.
- But this can be detrimental as well, since if a process starts hogging the RAM, other processes on the machine will get affected or might even crash. So we need a mechanism to help distribute these resources.
- This is where cgroups help us.

## Questions

- What user are you after chrooting and what permissions do you have ?
- How do permissions for parent child process relations ?

## Summary

<aside>
💡 chroot and namespace provides the needed isolation, whereas cgroups provides resource distribution.

</aside>

---

# Docker Basics

## Notes

### Images

- Images are pre-made containers. They basically have everything zipped and packed, and you need to just pull *(download)* and execute them. And then you will have a container
- Think of this, as those essential binaries and files that are necessary to run your service. Like we did for the `chroot` where it created a whole new root and we needed to move the binaries even for `bash` to access the command line, and `ls` and `cat` as well.
- So, images are nothing but snapshot of containers, with all the binaries and libs packed in.
- There are images available for running almost any library on the Docker Hub, which is nothing but a library for docker images. Think of it as npm packages for docker.
- Since images can go upto GB's they are stored as layers, where one image is built by building on top of *(layering)* multiple different images. This makes it lightweight to pull.
- Ex: For running the mongo container, you need to pull the mongo image which inturn relies on apache OS image, but since you already have apache OS image locally, only the extra needed images are pulled and stacked.
- Dangling image → Image which don't have links to any tagged images
- Unused image → image which is not assigned or used by a container
- Tagged image →  while building images, developer can assign some tag, which is nothing by an identification name/number provided to the image to track it.
- Tagged image → while building images, developer can assign some tag, which is nothing but an identification name/number provided to the image to help with tracking and versioning subsequent builds and deployments.

### Image commands ⇒

- `docker image ls`
- `docker image prune`
- `docker rmi <image_id>`

### Container ⇒

- When you unpack images, and execute them you get containers. They are the runtime abstraction of an image.
- You can view all docker containers using `docker ps -a` and view all the running ones using `docker ps`
- The difference between them is, that even after a container is killed they are still kept in memory, since one might need to analyze the logs of those containers. You need to explicitly remove them as well after killing them to remove every trace of container.
- Also, by default containers have non-persistent memory, so if you kill a container, all of the memory is lost. So, if you spin up a mongodb and then kill it, all of the database info is lost.
- One way to tackle that is using volumes, which we will explore later
- You can run containers in background, by using the `d` flag.
- You can attach to containers running in background, by using the `docker attach` command, followed by the container_id, which you can obtain from `docker ps`

### Container commands ⇒

- `docker run [OPTIONS] <image_name> [COMMAND] [ARG]`
- `docker logs`
- `docker ps`
- Among the options there are various flags for →
    - background
    - networking
    - memory mounts
    - port mapping
    - etc ...

### Dockerfile

- Dockerfile is used to containerize your application, and store it as an image.
- `docker build -t <image_name> <dockerfile_location>` This command, builds a docker image, by executing each instruction mentioned in dockerfile, step by step.
- This image, is a snapshot of your service, so if anyone else wants to spawn this service, or you need to spawn this service anywhere else you just build the image using Dockerfile, and then execute the image.
- A dockerfile can have multiple stages *(layers)* to optimize the final build size of images. So that it is more lightweight, flexible and portable.
- You can execute multiple distinct processes in a single container, but it is not recommended, as it makes debugging difficult and increases coupling as well with no added benefits. Ref - [https://docs.docker.com/config/containers/multi-service_container/](https://docs.docker.com/config/containers/multi-service_container/)
- Rather as recommended, you can run each service in their own container, and then orchestrate them using your own networks, and mounts. Docker-compose makes this all a breeze by helping you to set up multi-container projects.

### dockerignore

- At times you do not want to copy all files from your development folder into your dockerized container, dockerignore is used to make this smoother.
- Similar to gitignore, it completely ignores all the files and folders mentioned in dockerignore.
- Ex → I do not want to copy over the `.git/` folder as it might contain some critical data, and similarly I do not want to copy over  the `node_modules/` as it might contain some redundant and developer dependencies which are not needed for execution.
- Another reason for not having `node_modules` is, that your docker might have a different OS than your local host, and hence some dependencies which depend on OS, might not work if you copy directly, like - `node-sass`

### Some ideas

- Build your container using multiple layers, as this reduces your final image size.
    - Example, we do not need the whole nodejs ecosystem, including npm, we just need those dependencies, so we can generate those dependencies in a `node:12-stretch` container, which is a few hundred MB's in size. This is our first layer.
    - Next, we copy over these dependencies, to an alpine container, which is the smallest linux OS you can have, and install just nodejs manually from it's package manager. And then finally execute the files, since we have dependencies from our layer 1.
    - The final layer of alpine which is our layer 2, is barely 50 - 60 MB in size
    - Can inspect each individual layer as well. Ref - [https://stackoverflow.com/questions/42602731/can-i-run-an-intermediate-layer-of-a-docker-image](https://stackoverflow.com/questions/42602731/can-i-run-an-intermediate-layer-of-a-docker-image)
- Do not execute as root user in your containers, as it might introduces vulnerabilities. And there is no physical boundary between your container and the server. In case a security issue is find, having root permission makes it a cakewalk to exploit it.
- If using `EXPOSE` in Dockerfile to expose ports, be sure to check the mappings using `docker ps` . You can use the `--publish` or `-p` to define your own mappings.
- `COPY` is used for local files, `ADD` is used to copy from network locations as well.
- Some programs when run on docker, do not successfully intercept and process the `SIGTERM` command to kill the process, eg. `node` might become unresponsive at times, when trying to kill it in the docker. Hence, use the `init` flag, which is nothing but an initializer process, which handles the `SIGTERM` signal sent to the `node` or the spawned process.
- Another advantage of keeping your images small is that, small images have very few binaries, libs and applications running. Even if somehow, an attacker manages to sneak in a compromised python code, it does not have the python binaries to execute it. So, it is more secured by having less.

## Questions

- RUN vs CMD in Docker
    
    [RUN](https://docs.docker.com/engine/reference/builder/#run) is an image build step, the state of the container after a `RUN` command will be committed to the container image. A Dockerfile can have many `RUN` steps that layer on top of one another to build the image.
    
    [CMD](https://docs.docker.com/engine/reference/builder/#cmd) is the command the container executes by default when you launch the built image. A Dockerfile will only use the final `CMD` defined. The `CMD` can be overridden when starting a container with `docker run $image $other_command`.
    
    Ref - [https://stackoverflow.com/questions/37461868/difference-between-run-and-cmd-in-a-dockerfile](https://stackoverflow.com/questions/37461868/difference-between-run-and-cmd-in-a-dockerfile)
    

## Summary

<aside>
💡

</aside>

---

# Storage

## Notes

### Mounts ⇒

- Is nothing but adding different memory types to your docker container.
- Note that, these are attached to docker containers and not images.
- If you run docker containers without any mounts attached, when the docker container is killed, all the generated data is also lost with it. In short docker uses non-persistent storage by default, which doesn't suit well for use cases such as databases.
- There are various types of mounts depending on your use-case → bind mount, volumes, tempfs

### Bind mounts

- are used to bind your local host data, to your container data.  It opens up a tunnel from your local host mount, to the container mount.
- In the below example, `target` folder inside the present working directory from local host, is tunneled to `/app` directory in the container.

```bash
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```

- There can be more options added such as `ro` which define a read only state.
- Very useful to setup local development environments, where your application files are run locally, and a demon process executes those files inside the container, so on file changes, they are automatically restarted.

### Volumes ⇒

- these are completely managed by docker, and do not depend on the host machine OS or their directory structure. There are lots of benefits of volumes over bind mounts →
- Volumes are easier to back up or migrate than bind mounts.
- You can manage volumes using Docker CLI commands or the Docker API.
- Volumes work on both Linux and Windows containers.
- Volumes can be more safely shared among multiple containers.
- Volume drivers let you store volumes on remote hosts or cloud providers, to encrypt the contents of volumes, or to add other functionality.

Ref - [https://docs.docker.com/storage/volumes/](https://docs.docker.com/storage/volumes/)

- They are ideal when you need to share persistent storage between several docker containers. Example → For storing data of a database.
- In the below CLI a volume is created, named as - `myvol2` and it is mounted at the location `/app` in the container.

```bash
$ docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

- There are other options such as `ro` which can be provided.

## Questions

- How to access and inspect data in volumes, locally
    - Volumes are created and stored in your local file system.
    - The location might be specific from OS to OS.
    - As can be seen below, volume is created, *(yellow highlighted)* in the local file system.
    - Ref - [https://forums.docker.com/t/how-to-access-docker-volume-data-from-host-machine/88063/4](https://forums.docker.com/t/how-to-access-docker-volume-data-from-host-machine/88063/4)
    
    ![Intro%20to%20Containers%20a43bef07138445878431add2510b56ff/Untitled%201.png](Intro%20to%20Containers%20a43bef07138445878431add2510b56ff/Untitled%201.png)
    

## Summary

<aside>
💡

</aside>

---

# Networking

## Notes

### Networks

- At times you will need to communicate between various services sitting on different containers, how do you make them talk ? You use docker networks !
- As with mounts, there are various types of docker networks depending on the use-case you will use one.

### Bridge

- As the name suggests it acts a bridge between containers, which are connected to the same bridge and isolates from rest of the network bridges which it is not connected to.
- Bridge networks only apply to the containers running on the **same** docker daemon. So, if you have containers running on different hosts orchestrated via swarm or kubernetes, you will need *overlay network* for this.
- It is a Link Layer device which forwards traffic between network segments.

### Host

- The network stack of container **is not** isolated from the host's network stack. The container shares the hosts's networking namespace. It is useful when there are high performance needs, like → large range of ports need to be handled, since it does not require NAT to translate IP between containers and host.

### None

- Disable all networking. Cannot talk to anyone else.

## Questions

- A is connected to B via network bridge n1, A is connected to C via network bridge n2, can B and C still communicate as well ?

## Summary

<aside>
💡

</aside>

---

# Docker Arch Overview

## Notes

- Mainly consists of three components ⇒ client, daemon *(server)* and registry

### Docker Client

- Docker Client, interacts with Docker Daemon via REST API.
- Primary way Docker users interact with Docker. When you use commands such as `docker run` on the CLI, you are interacting with the Docker Client.
- Or when you click the `Start` icon, on the Docker Desktop app.
- This is when the Docker Client sends an API call to the Docker Daemon, who executes that command.
- Can communicate with more than one docker daemon, either on the host machine, local network or somewhere else.

### Docker Daemon

- Listens for API calls from Docker Client, and manages the Docker objects, like images, containers, network, volumes, etc ...
- API calls can be made via unix socket, tcp or fd calls, depending on where the daemon and the client resides.

### Docker Registry

- Registry stores all Docker images, akin to npm for node packages. This is the public docker registry.
- Can create your own private docker registry as well, to manage your own images without showcasing it to the outside world.
- Can use other managed docker registries as well like - GHCR, or [quay.io](http://quay.io)

## Summary

<aside>
💡

</aside>

---

# OCI

## Notes

### OCI

- The Open Container Initiative is an open governance structure for the express purpose of creating open industry standards around container formats and runtimes.
- It currently defines two specs ⇒
    1. Runtime Spec → Outlines how to run a “filesystem bundle” that is unpacked on disk. At a high-level an OCI implementation would download an OCI Image then unpack that image into an OCI Runtime filesystem bundle.
    2. Image Spec → Outlines how container images should be created
- Docker is a part of OCI, and so are other tools like Buildah, Podman, among others.
- Hence, they can interoperated, example → you can run docker containers, on buildah, and you can build containers using buildah and run on docker. Each has their own pros and cons.

## Summary

<aside>
💡

</aside>

---

# References ⇒

1. [https://btholt.github.io/complete-intro-to-containers/](https://btholt.github.io/complete-intro-to-containers/) 
2. [https://www.netapp.com/blog/containers-vs-vms/](https://www.netapp.com/blog/containers-vs-vms/)
3. [https://www.redhat.com/sysadmin/7-linux-namespaces](https://www.redhat.com/sysadmin/7-linux-namespaces)
4. [https://dockerlabs.collabnix.com/beginners/components/container-vs-image.html](https://dockerlabs.collabnix.com/beginners/components/container-vs-image.html)
5. [https://docs.docker.com/engine/reference/commandline/run/](https://docs.docker.com/engine/reference/commandline/run/)
6. [https://docs.docker.com/config/containers/multi-service_container/](https://docs.docker.com/config/containers/multi-service_container/)
7. [https://stackoverflow.com/questions/37461868/difference-between-run-and-cmd-in-a-dockerfile](https://stackoverflow.com/questions/37461868/difference-between-run-and-cmd-in-a-dockerfile)
8. [https://docs.docker.com/engine/reference/commandline/run/](https://docs.docker.com/engine/reference/commandline/run/)
9. [https://docs.docker.com/storage/bind-mounts/](https://docs.docker.com/storage/bind-mounts/)
10. [https://docs.docker.com/storage/volumes/](https://docs.docker.com/storage/volumes/)
11. [https://forums.docker.com/t/how-to-access-docker-volume-data-from-host-machine/88063/4](https://forums.docker.com/t/how-to-access-docker-volume-data-from-host-machine/88063/4)
12. [https://docs.docker.com/network/](https://docs.docker.com/network/)
13. [https://docs.docker.com/get-started/overview/#docker-architecture](https://docs.docker.com/get-started/overview/#docker-architecture)
14.