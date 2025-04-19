---
title : "Docker Essentials: From Installation to Custom Images"
description: Learn Docker basics, build custom images, and deploy containerized apps with ease.
author : k4n3ki
date : 2025-04-19 1:00:00 +530
categories: [DevOps]
tags: [Docker, Container, DevOps]
image:
    path: /assets/img/dockerTutorial/docker.png
    alt: Docker logo
---


A few weeks ago, I began my DevOps journey through the [DevOps Mastery Specialization](https://www.coursera.org/specializations/devops-mastery) on Coursera by KodeKloud—a fantastic starting point for anyone new to this field. Last week, I published a blog post on [Jenkins](https://0xk4n3ki.github.io/posts/JenkinsTutorial/), where I covered its installation, initial setup, and how to create a basic pipeline. In this blog, I’ll be diving into another essential DevOps tool—[Docker](https://www.docker.com/). I’ll walk through the installation process, introduce commonly used commands, demonstrate how to create a custom Docker image, and explain how to push it to remote repositories like Docker Hub.

## <span style = "color:red;">Content</span>
- [Introduction](#introduction)
- [Installing Docker](#installing-docker)
- [Key Concepts](#key-concepts)
- [Basic Docker Commands](#basic-docker-commands)
- [Creating a Custom Docker Image](#creating-a-custom-docker-image)
- [Running the Custom Image](#running-the-custom-image)
- [Creating a multistage Dockerfile](#creating-a-multistage-dockerfile)
- [Tagging and Pushing to Docker Hub](#tagging-and-pushing-to-docker-hub)
- [Common Issues and Troubleshooting](#common-issues-and-troubleshooting)
- [Conclusion](#conclusion)
- [References / Resources](#references--resources)


## <span style = "color:red;">Introduction</span>

### <span style = "color:lightgreen;">What is Docker?</span>
Docker is an open-source platform that enables developers to build, package, deploy, and manage applications using containers. It provides a consistent environment by packaging an application along with all its dependencies into a single unit called a Docker image. This image can then be used to run containers—lightweight, isolated instances of applications that behave the same way across different systems.

Unlike traditional deployments where dependencies must be installed manually on each machine, Docker ensures everything the application needs (libraries, runtime, environment variables, etc.) is already included within the image, eliminating "it works on my machine" problems.

### <span style = "color:lightgreen;">Why Use Docker?</span>

1. **Lightweight and Efficient**:
Containers are much more lightweight than virtual machines. While VMs simulate an entire operating system along with hardware through a hypervisor, Docker containers share the host OS kernel and only run the necessary libraries and binaries. For example, if you run a container using an Ubuntu base image, it won’t have systemctl or systemd since it's designed to run just a single isolated process—not an entire OS.

2. **Portability**:
Docker containers can run on any system that supports Docker—whether it's Linux, macOS, Windows, or a cloud provider. This allows developers to write code once and run it anywhere without worrying about environment inconsistencies.

3. **Speed and Scalability**:
Because containers are lightweight and don’t require booting up an entire OS, they start almost instantly. Developers can spin up multiple instances of an application in seconds, which is ideal for scaling.

4. **Version Control and Rollback**:
Docker tracks versions of images, making it easy to roll back to previous builds. This is especially useful in CI/CD pipelines where consistency and control are crucial.

5. **Ecosystem and Community**:
Docker has a vast open-source ecosystem. Developers can access Docker Hub—a public registry with thousands of pre-built container images contributed by the community and official sources.

### <span style = "color:lightgreen;">Real-World use cases</span>

- **Microservices architecture:** Run isolated services for better scalability and maintainability.
- **Reproducible development environments:** Mirror production to avoid "it works on my machine" problems.
- **CI/CD pipelines:** Automate builds, tests, and deployments in containerized workflows.
- **Cloud-native apps:** Deploy seamlessly across AWS, Azure, or GCP.
- **Legacy app modernization:** Wrap legacy software in containers for portability and ease of management.

## <span style = "color:red;">Installing Docker</span>

### <span style = "color:lightgreen;">Linux (Ubuntu - Preferred)</span>

For Ubuntu, you only need to install the Docker Engine, as the Linux kernel natively supports containerization. Below are the recommended steps to install Docker using Docker’s official APT repository:

Step 1: Set up the Docker APT Repository

```bash
# Update package index and install required packages
sudo apt-get update
sudo apt-get install ca-certificates curl

# Create directory for Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings

# Download and store Docker's official GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the Docker's repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package index again
sudo apt-get update
```

Step 2: Install Docker


```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Step 3: Verify Docker Installation

Run the official hello-world container to verify that Docker is installed correctly:
```bash
sudo docker run hello-world
```

If everything is working, you'll see a message confirming that Docker is set up correctly.

<img alt="hello world container" src="/assets/img/dockerTutorial/hello.png">


### <span style = "color:lightgreen;">Windows & macOS</span>

Unlike Linux, Windows and macOS do not have a native Linux kernel, which Docker relies on. Therefore, Docker runs inside a lightweight virtual machine using:

- WSL2 (Windows Subsystem for Linux 2) on Windows
- HyperKit on macOS

To install Docker on these platforms:

- Download Docker Desktop from the official site: https://docs.docker.com/desktop/
- Follow the installation instructions specific to your OS.
- After installation, verify it by opening a terminal (or PowerShell) and running:
```bash
docker run hello-world
```

Docker Desktop comes bundled with everything needed, including the Docker Engine, Docker CLI, and Docker Compose plugin.


## <span style = "color:red;">Key Concepts</span>

### <span style = "color:lightgreen;">Image vs Container</span>

Image: </br>
A Docker image is a lightweight, standalone, and immutable template used to create containers. Think of it like a virtual machine snapshot or blueprint. It includes everything needed to run an application—code, runtime, libraries, environment variables, and configuration files.

Container: </br>
A container is a running instance of an image. It is an isolated environment that runs a set of processes with its own filesystem, networking, and resources—created from a Docker image. Multiple containers can be spun up from the same image, each running independently.

### <span style = "color:lightgreen;">Docker Engine</span>

Docker Engine is the core component that enables containerization. It is an open-source container runtime that acts as a client-server application with three main components:
- dockerd (Daemon): A long-running background process responsible for managing containers, images, volumes, and networks.
- REST API: Provides programmatic access to Docker’s features, allowing tools and scripts to interact with the daemon.
- Docker CLI (docker): A command-line interface that sends commands to the Docker daemon using the REST API.

### <span style = "color:lightgreen;">Docker Hub</span>

Docker Hub is a cloud-based container registry where developers can store, share, and manage Docker images. It hosts both public and private repositories. You can:
- Pull pre-built images from official sources or community contributors.
- Push your own custom images for versioning or sharing with teams.

### <span style = "color:lightgreen;">Dockerfile</span>

A Dockerfile is a script-like text file that defines the steps to build a Docker image. It acts as a blueprint for the image, specifying:
- The base image to start from
- Files to copy
- Packages to install
- Commands to run
- Default working directory and entry point

By running `docker build` on a directory with a Dockerfile, Docker constructs a new image as per the instructions.

### <span style = "color:lightgreen;">Volumes</span>

Volumes are Docker’s preferred mechanism for persisting data generated and used by containers. Unlike the container’s writable layer (which is ephemeral), volumes are stored on the host filesystem—typically at /var/lib/docker/volumes on Linux.

Key benefits:
- Data persists across container restarts and removals
- Volumes can be shared across multiple containers
- They’re managed entirely by Docker and are more efficient than using bind mounts for production use

## <span style = "color:red;">Basic Docker Commands</span>

### <span style = "color:lightgreen;">docker run</span>

The `docker run` command is used to create and start a new container. It first checks if the specified image exists locally. If not, Docker automatically pulls the image from [Docker Hub](https://hub.docker.com/).

In the example below, the command runs an Ubuntu 20.04 container with bash:

```bash
docker run -it --name my_ubuntu_container ubuntu:20.04 bash
```
- `-i`: Runs the container in interactive mode.
- `-t`: Allocates a pseudo-TTY for the container, giving it terminal access.
- `--name`: Assigns a custom name to the container; otherwise, Docker generates a random name.
- `-d`: (Optional) Runs the container in detached mode (in the background).

<img alt="docker run" src="/assets/img/dockerTutorial/run.png">

### <span style = "color:lightgreen;">docker ps -a</span>

- `docker ps`: Lists only running containers.
- `docker ps -a`: Lists all containers, including stopped (exited) ones.

The output includes:
- Container ID: Used to reference the container in other commands.
- Image: The image used to create the container.
- Command: The process running inside the container.
- Name: The container’s unique name.

<img alt="docker ps" src="/assets/img/dockerTutorial/ps.png">

### <span style = "color:lightgreen;">docker stop</span>

The `docker stop` command stops a running container by gracefully shutting down its main process.

In the example below, a container runs Ubuntu with a sleep 1000 command in detached mode:

```bash
docker run -d --name ubuntu_sleep ubuntu:20.04 sleep 1000
```

<img alt="sleep" src="/assets/img/dockerTutorial/sleep.png">

We can confirm it’s running with:

```bash
docker ps
```

<img alt="sleep ps" src="/assets/img/dockerTutorial/sleep_ps.png">

To stop it:

```bash
docker stop ubuntu_sleep
```

<img alt="sleep stop" src="/assets/img/dockerTutorial/sleep_stop.png">

After stopping, `docker ps -a` shows the container’s status as Exited:

<img alt="sleep ps -a" src="/assets/img/dockerTutorial/sleep_psa.png">

### <span style = "color:lightgreen;">docker start</span>

The `docker start` command is used to restart a stopped (exited) container. It uses the container's name or ID.

For example, suppose you had created a file inside the container before stopping it:

```bash
docker exec -it my_ubuntu_20.04 bash
echo "learning Docker" > ~/file.txt
```

<img alt="start setup" src="/assets/img/dockerTutorial/start_setup.png">

After stopping the container, you can resume it:

```bash
docker start -ai my_ubuntu_20.04
```
- `-a`: Attach STDOUT/STDERR.
- `-i`: Keep STDIN open.

This brings the container back online with all its previously saved state.

<img alt="stat" src="/assets/img/dockerTutorial/start.png">

### <span style = "color:lightgreen;">docker rm</span>

The `docker rm` command is used to delete an exited container from the local host. It requires either the container's name or ID as an argument. Running containers must be stopped before they can be removed.

### <span style = "color:lightgreen;">docker images</span>

The docker images command lists all locally available images that have been pulled or built. The output includes:
- `REPOSITORY`: Name of the image (e.g., ubuntu)
- `TAG`: Version tag (e.g., latest)
- `IMAGE ID`: Unique identifier for the image
- `CREATED`: Time when the image was created
- `SIZE`: Size of the image on disk

<img alt="docker images" src="/assets/img/dockerTutorial/images.png">


### <span style = "color:lightgreen;">docker rmi</span>

The `docker rmi` command removes a Docker image from the local system. It accepts either the image name with tag (e.g., ubuntu:20.04) or the image ID.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> An image cannot be removed if any containers (even exited ones) still reference it. Those containers must be deleted first using docker rm.
{: .prompt-info }
<!-- markdownlint-restore -->

<img alt="docker rmi" src="/assets/img/dockerTutorial/rmi.png">

### <span style = "color:lightgreen;">docker pull</span>

The `docker pull` command downloads a Docker image from a remote repository (e.g., Docker Hub, GitHub Container Registry) without running it. This is useful for pre-fetching images for later use or deployment.

### <span style = "color:lightgreen;">docker exec</span>

The docker exec command is used to run a command inside an actively running container. It allows interactive debugging or inspection of container internals.

Example:
```bash
docker exec ubuntu_sleep cat /etc/os-release
```
This command prints the OS release information inside the ubuntu_sleep container.

<img alt="docker exec" src="/assets/img/dockerTutorial/exec.png">


## <span style = "color:red;">Creating a Custom Docker Image</span>

To create a Docker image, a file named Dockerfile must be created. This file contains instructions that define how the image should be built, including the base image, necessary dependencies, application files, and the command to execute when the container starts.

For demonstration, consider a basic Flask application that listens on port 8081 and returns a simple message:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Application inside Docker\n"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8081)
```

To containerize this application, a lightweight Python image (e.g., python:3.9-slim) will be used as the base image. The Flask module must be installed, the application file copied into the container, and port 8081 exposed for external access.

The corresponding Dockerfile is:

```Dockerfile
# Use an official lightweight Python image
FROM python:3.9-slim   

# Set the working directory
WORKDIR /app  

# Copy the Flask app into the container
COPY app.py .  

# Install Flask
RUN pip install flask

# Expose port 8081 to the host
EXPOSE 8081  

# Command to run the application
CMD ["python", "app.py"]
```

To build the image, execute the following command in the directory containing the Dockerfile:

```bash
docker build . -t flask_app
```

<img alt="docker build" src="/assets/img/dockerTutorial/build.png">

This command creates an image named flask_app.

## <span style = "color:red;">Running the Custom Image</span>

To run a container from this image and expose it on port 8081, use:

```bash
docker run -p 8081:8081 flask_app
```

- The -p flag maps the host port to the container port, allowing access to the application from outside the container.

<img alt="app run" src="/assets/img/dockerTutorial/app_run.png">

To verify that the containerized application is running, use a curl command or visit http://127.0.0.1:8081 in a web browser:

```bash
curl http://127.0.0.1:8081
```

<img alt="curl" src="/assets/img/dockerTutorial/curl.png">


> now, for multistage and volume mapping, use the peparser image

## <span style = "color:red;">Creating a multistage Dockerfile</span>

The `PEparser` application, written in C++, analyzes 32-bit Portable Executable (PE) files. It uses the Windows runtime environment and requires Wine to run on a Linux-based system. The application is compiled using the mingw-w64 cross-compiler.

The compilation command used is:

```bash
x86_64-w64-mingw32-g++ PEparser.cpp -o pe.exe -fpermissive -Wint-to-pointer-cast
```

### <span style = "color:lightgreen;">Initial Single-Stage Dockerfile (Large Image Size)</span>

Initially, a single-stage Dockerfile was used to both build and run the application. This approach resulted in a large image size (approximately 1.5 GB) due to the inclusion of both build tools and runtime dependencies within the same image.

```Dockerfile
FROM ubuntu

RUN dpkg --add-architecture i386
RUN apt-get update
RUN export DEBIAN_FRONTEND=noninteractive
RUN apt-get install -y --no-install-recommends build-essential mingw-w64 wine64 && rm -rf /var/lib/apt/lists/*

COPY ./peparser.cpp /opt/peparser.cpp

RUN x86_64-w64-mingw32-g++ /opt/peparser.cpp -o /opt/peparser.exe -fpermissive -Wint-to-pointer-cast -static-libgcc -static-libstdc++

ENTRYPOINT ["wine", "/opt/peparser.exe"]
```

This Dockerfile installs both the compiler toolchain and the Windows runtime environment, even though only the latter is necessary at runtime.

### <span style = "color:lightgreen;">Optimized Multi-Stage Dockerfile</span>

To reduce image size, a **multi-stage build** was adopted. The application is first compiled in a build stage, and only the final executable is passed to a second, minimal runtime stage that includes Wine.

```Dockerfile
# stage 1
FROM ubuntu AS build

RUN dpkg --add-architecture i386
RUN apt-get update
RUN export DEBIAN_FRONTEND=noninteractive
RUN apt-get install -y --no-install-recommends build-essential mingw-w64

COPY ./peparser.cpp /opt/peparser.cpp

RUN x86_64-w64-mingw32-g++ /opt/peparser.cpp -o /opt/peparser.exe -fpermissive -Wint-to-pointer-cast -static-libgcc -static-libstdc++

# stage 2
FROM ubuntu

RUN apt-get update
RUN apt-get install -y --no-install-recommends wine && rm -rf /var/lib/apt/lists/*

COPY --from=build /opt/peparser.exe /opt/peparser.exe

ENTRYPOINT ["/usr/bin/wine", "/opt/peparser.exe"]
```

This approach reduces the final image size to approximately 901 MB. Further optimization is possible (e.g., using Alpine-based images with Wine support), but this serves as a substantial improvement.

### <span style = "color:lightgreen;">Running the Container with Input Files</span>

To analyze PE files located on the host system, volume mounting is required using the -v flag. This allows the container to access files from the host and persist any changes:

```bash
docker run -it -v <host-dir>:<container-dir> peparser_image
```

- The `-v` flag maps a host directory to a directory within the container.
- This enables the containerized application to read PE files from the host and write output or logs back to the mapped directory.
- Data stored in the mapped volume remains persistent even after the container exits or is removed.

## <span style = "color:red;">Tagging and Pushing to Docker Hub</span>

Tagging Docker images is considered best practice for purposes such as version control, image differentiation, and environment-specific deployments. Tags help track changes, manage multiple builds, and maintain clear documentation of an image’s purpose or state (e.g., development, staging, or production).

For example, in the case of the PEparser image, the following naming convention was used:

```bash
ghcr.io/0xk4n3ki/peparser:multi-stage-build
```

- ghcr.io: Specifies the GitHub Container Registry as the remote repository.
- 0xk4n3ki: The GitHub username or organization name under which the image will be hosted.
- peparser: The image name.
- multi-stage-build: The tag, which could also follow semantic versioning (e.g., v1.0, v1.1, latest) for clarity.

To push the image to the GitHub Container Registry, the following command is used:

```bash
docker push ghcr.io/0xk4n3ki/peparser:multi-stage-build
```

Before pushing, authentication with the container registry (in this case, `ghcr.io`) may be required. This typically involves generating a personal access token with the appropriate `write:packages` and `read:packages` permissions, then logging in using:

```bash
echo <TOKEN> | docker login ghcr.io -u <USERNAME> --password-stdin
```

If the image is to be pushed to Docker Hub, replace the registry URL with the Docker Hub namespace and ensure authentication using:

```bash
docker login
```

Tag the image accordingly:

```bash
docker tag peparser username/peparser:multi-stage-build
docker push username/peparser:multi-stage-build
```

Proper tagging ensures reproducibility, facilitates continuous integration workflows, and enables better organization of image versions across environments and platforms.


## <span style = "color:red;">Common Issues and Troubleshooting</span>

### <span style = "color:lightgreen;">Docker daemon not running</span>

Docker CLI commands require the Docker daemon to be active. If errors like Cannot connect to the Docker daemon appear, ensure the service is running using:

```bash
sudo systemctl start docker
```

### <span style = "color:lightgreen;">Port conflicts</span>

Running containers with exposed ports that are already in use can lead to binding errors. Verify which ports are open using `netstat`, `lsof`, or `ss`, and either free the port or run the container on a different one using the `-p` flag.

### <span style = "color:lightgreen;">Container exit codes</span>

Containers may exit immediately after starting if the main process ends or encounters an error. Check the container's logs using:

```bash
docker logs <container_name_or_id>
```

Inspect the exit code using:

```bash
docker inspect <container_name_or_id> --format='{{.State.ExitCode}}'
```

Non-zero exit codes often indicate misconfigured startup commands or missing dependencies.


## <span style = "color:red;">Conclusion</span>

This guide covered the Docker installation, essential commands, image lifecycle, creation of custom application images, multi-stage builds, and pushing images to remote registries like Docker Hub and GitHub Container Registry.

Understanding these fundamentals enables smoother integration into real-world development and DevOps workflows. The next logical steps include:

- Using Docker in CI/CD pipelines
- Deploying containers with Kubernetes
- Exploring Docker Compose for multi-container applications
- Understanding Docker networking and volumes for stateful services


## <span style = "color:red;">References / Resources</span>

<span style="color:lightgreen">Docker Documentation</span>
- [https://docs.docker.com/](https://docs.docker.com/)
- [https://docs.docker.com/desktop/setup/install/linux/ubuntu/](https://docs.docker.com/desktop/setup/install/linux/ubuntu/)
- [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

<span style="color:lightgreen">Docker playground</span>
- [https://labs.play-with-docker.com/](https://labs.play-with-docker.com/)

<span style="color:lightgreen">Github container Repository</span>
- [https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

<span style="color:lightgreen">GFG flask app</span>
- [https://www.geeksforgeeks.org/dockerize-your-flask-app/](https://www.geeksforgeeks.org/dockerize-your-flask-app/)

<span style="color:lightgreen">Kodekloud Course</span>
- [https://www.coursera.org/specializations/devops-mastery](https://www.coursera.org/specializations/devops-mastery)
- [https://www.coursera.org/learn/docker-basics-for-devops/](https://www.coursera.org/learn/docker-basics-for-devops/)