# Docker, Containerization, Virtualization, Architecture and Core Components

# 1. What is Virtualization?

Virtualization is a technology that allows us to create multiple **virtual machines (VMs)** on a single physical server.

A software layer called a **hypervisor** manages these virtual machines and provides each VM with virtual CPU, memory, storage, and networking.

Each virtual machine typically contains:

- A complete guest operating system
- Application runtime
- Libraries and dependencies
- Application

### Example

Suppose we have one physical server:

```text
Physical Server
      |
      v
   Hypervisor
   /          VM1       VM2
   |         |
Linux      Linux
   |         |
App A      App B
```

Each VM has its own operating system.

---

# 2. Types of Virtualization

There are two common types of hypervisors.

## Type 1 Hypervisor

A Type 1 hypervisor runs directly on the physical hardware.

```text
Physical Hardware
       |
       v
Type 1 Hypervisor
   /           VM1          VM2
```

Examples include:

- VMware ESXi
- Microsoft Hyper-V
- Xen

Type 1 hypervisors are commonly used in data centers and cloud environments.

## Type 2 Hypervisor

A Type 2 hypervisor runs on top of a host operating system.

```text
Physical Hardware
       |
       v
Host OS
       |
       v
Type 2 Hypervisor
   /           VM1          VM2
```

Examples include:

- VMware Workstation
- Oracle VirtualBox
- Parallels Desktop

---

# 3. Benefits of Virtualization

Virtualization provides several benefits:

- Better hardware utilization
- Server consolidation
- Isolation between workloads
- Easy provisioning of virtual machines
- Support for multiple operating systems on one physical server
- Snapshot and cloning capabilities
- Easier disaster recovery
- Flexible resource allocation
- Reduced physical hardware requirements

---

# 4. What is Containerization?

Containerization is a method of packaging an application together with its required libraries, dependencies, configuration, and runtime environment into an isolated unit called a **container**.

Unlike a traditional VM, a container does not normally include a complete guest operating system.

Containers share the host operating system kernel while remaining isolated from each other.

```text
Physical / Virtual Server
        |
        v
   Host Operating System
        |
        v
Container Runtime
   |       |       |
   v       v       v
Container Container Container
   |       |       |
 App A    App B    App C
```

---

# 5. Why Do We Need Containerization?

Before containers, developers often faced the problem:

> "It works on my machine, but it doesn't work in production."

This can happen because of differences in:

- Operating system
- Library versions
- Runtime versions
- Configuration
- Dependencies
- Environment variables

Containerization packages the application and its required environment together.

```text
Application
    +
Dependencies
    +
Libraries
    +
Runtime
    +
Configuration
        |
        v
    Container
```

This makes application deployment more consistent.

---

# 6. Benefits of Containerization

### 1. Portability

The same container image can be used across different environments that support the required container runtime.

```text
Developer Laptop
       |
       v
     Image
       |
       +------> Test
       |
       +------> QA
       |
       +------> Production
```

### 2. Lightweight

Containers generally do not include a complete guest operating system.

### 3. Fast Startup

Containers can usually start much faster than full virtual machines.

### 4. Isolation

Applications can run in isolated environments.

### 5. Consistency

The same image can be promoted across development, testing, and production environments.

### 6. Efficient Resource Usage

Multiple containers can share the host operating system kernel.

### 7. Scalability

Containers can be created and removed quickly, which is useful for microservices and cloud-native applications.

---

# 7. Virtualization vs Containerization

| Feature | Virtualization | Containerization |
|---|---|---|
| Unit | Virtual Machine | Container |
| Operating System | Each VM normally has its own guest OS | Containers share the host kernel |
| Size | Usually larger | Usually smaller |
| Startup | Usually slower | Usually faster |
| Resource usage | Higher | Lower |
| Isolation | Strong VM-level isolation | Process-level/application isolation |
| Portability | Portable VM images | Portable container images |
| Typical use | Full OS workloads | Applications and microservices |

### Visual Comparison

```text
Virtualization
----------------------------

Physical Server
       |
       v
   Hypervisor
   /          VM1       VM2
  |          |
Guest OS   Guest OS
  |          |
 App A      App B
```

```text
Containerization
----------------------------

Physical / Virtual Server
       |
       v
    Host OS
       |
       v
Container Runtime
   /      |        C1      C2      C3
  |       |       |
 App A   App B   App C
```

---

# 8. What is Docker?

Docker is a platform used to **build, package, distribute, and run applications as containers**.

Docker provides tools and components that make containerization easier.

A simple Docker workflow is:

```text
Application Source Code
          |
          v
      Dockerfile
          |
          v
     docker build
          |
          v
      Docker Image
          |
          v
      docker run
          |
          v
     Docker Container
```

Docker is widely used for:

- Application packaging
- Microservices
- CI/CD
- Development environments
- Testing
- Cloud-native applications
- DevOps workflows
- Kubernetes workloads

---

# 9. What Problem Does Docker Solve?

Without containers:

```text
Developer Environment
       |
       | Different dependencies
       v
Testing Environment
       |
       | Different configuration
       v
Production
```

This can cause environment-related problems.

With Docker:

```text
Source Code
    |
    v
Docker Image
    |
    +------> Development
    |
    +------> Testing
    |
    +------> Production
```

The same image can be used throughout the application lifecycle.

---

# 10. Docker Image

A Docker image is a **read-only, immutable template used to create containers**.

An image contains the application and everything required to run it, such as:

- Application code
- Libraries
- Dependencies
- Runtime
- Required filesystem content
- Configuration defaults

Example:

```text
Docker Image
    |
    +-- Base OS/filesystem
    +-- Runtime
    +-- Libraries
    +-- Application
    +-- Configuration
```

### Example

A Python application image might contain:

```text
Python Image
    +
Python Dependencies
    +
Application Code
        |
        v
   Docker Image
```

---

# 11. How is a Docker Image Created?

Usually, a Dockerfile is used.

Example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Build:

```bash
docker build -t my-python-app .
```

This creates an image:

```text
my-python-app
```

---

# 12. Docker Container

A Docker container is a **running instance of a Docker image**.

For example:

```bash
docker run nginx
```

Docker uses the Nginx image to create and start a container.

Conceptually:

```text
Docker Image
     |
     | docker run
     v
Docker Container
```

An image is the template.

A container is the running instance.

---

# 13. Image vs Container

| Docker Image | Docker Container |
|---|---|
| Read-only template | Running instance of an image |
| Used to create containers | Created from an image |
| Immutable | Has a writable container layer |
| Stored locally or in a registry | Runs on a Docker host |
| Can create multiple containers | Represents a running/stopped workload |

Example:

```text
              nginx Image
                  |
        +---------+---------+
        |         |         |
        v         v         v
    Container  Container  Container
       1          2          3
```

One image can be used to create multiple containers.

---

# 14. Dockerfile

A Dockerfile is a text file containing instructions used to build a Docker image.

Example:

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Common Dockerfile instructions include:

```text
FROM
WORKDIR
COPY
ADD
RUN
ARG
ENV
EXPOSE
CMD
ENTRYPOINT
USER
VOLUME
```

---

# 15. Docker Registry

A Docker registry is a repository where Docker images can be stored and distributed.

Examples:

- Docker Hub
- Amazon Elastic Container Registry (ECR)
- Azure Container Registry (ACR)
- Google Artifact Registry
- GitHub Container Registry

Basic flow:

```text
Developer
    |
    | docker build
    v
Docker Image
    |
    | docker push
    v
Container Registry
    |
    | docker pull
    v
Server / Kubernetes
```

---

# 16. Docker Hub

Docker Hub is a public/private container image registry provided by Docker.

For example:

```bash
docker pull nginx
```

Docker can pull the Nginx image from a configured registry, commonly Docker Hub when no other registry is specified.

---

# 17. Docker Engine

Docker Engine is the core technology used to build and run containers.

At a high level, it includes components responsible for:

- Docker API
- Image management
- Container lifecycle
- Networking
- Storage
- Container execution

Modern Docker uses the **Moby project** and **containerd/runc-based components** underneath the Docker Engine architecture.

---

# 18. Docker Architecture

A simplified Docker architecture looks like this:

```text
                 Docker Client
                      |
                      | Docker API
                      v
              Docker Engine
              /     |                   /      |               Images   Containers  Networks
           |         |          |
           |         |          |
           +---------+----------+
                     |
                   Storage
```

A more detailed view:

```text
User
 |
 | docker build
 | docker run
 | docker pull
 | docker push
 v
+----------------------+
|    Docker Client     |
|       (CLI)          |
+----------+-----------+
           |
           | Docker API
           v
+----------------------+
|    Docker Engine     |
|                      |
|  Image Management    |
|  Container Lifecycle |
|  Network Management  |
|  Volume Management   |
+----------+-----------+
           |
           v
      containerd
           |
           v
         runc
           |
           v
      Linux Kernel
```

The exact internal architecture can vary by platform and Docker version, but this is a useful conceptual model.

---

# 19. Docker Client

The Docker Client is the command-line interface used by users and automation tools to interact with Docker.

Examples:

```bash
docker build
docker run
docker ps
docker images
docker pull
docker push
docker network
docker volume
```

When you execute:

```bash
docker run nginx
```

the Docker CLI sends the request to the Docker Engine.

---

# 20. Docker Daemon / Docker Engine

Historically, Docker used the term **Docker daemon (`dockerd`)** for the long-running background process.

It is responsible for handling Docker API requests and managing:

- Images
- Containers
- Networks
- Volumes
- Container lifecycle

A simplified view:

```text
Docker CLI
    |
    v
Docker Engine / dockerd
    |
    +---- Images
    +---- Containers
    +---- Networks
    +---- Volumes
```

---

# 21. containerd

`containerd` is a container runtime component responsible for container lifecycle management.

It handles responsibilities such as:

- Image management
- Container lifecycle
- Container execution coordination
- Snapshot/filesystem management

Conceptually:

```text
Docker Engine
      |
      v
  containerd
      |
      v
     runc
      |
      v
Container
```

---

# 22. runc

`runc` is a low-level OCI-compliant runtime used to create and run containers.

At a high level:

```text
Docker
  |
  v
containerd
  |
  v
runc
  |
  v
Linux Kernel
  |
  v
Container Process
```

---

# 23. Linux Kernel and Container Isolation

Containers share the host kernel but use Linux kernel features to provide isolation.

Important mechanisms include:

### Namespaces

Namespaces isolate things such as:

- Process IDs
- Network
- Mounts/filesystems
- Hostnames
- Users

### Control Groups (cgroups)

cgroups control and limit resources such as:

- CPU
- Memory
- PIDs
- Other resource usage

Conceptually:

```text
Linux Kernel
    |
    +---- Namespaces
    |       |
    |       +-- Process isolation
    |       +-- Network isolation
    |       +-- Filesystem isolation
    |
    +---- cgroups
            |
            +-- CPU limits
            +-- Memory limits
```

---

# 24. Docker Volume

A Docker volume is a mechanism for storing persistent data outside the writable layer of a container.

Why is this needed?

Containers are designed to be replaceable and disposable.

If a container is removed:

```bash
docker rm mycontainer
```

data stored only inside its writable container layer can be lost.

Volumes provide persistent storage.

```text
Container
    |
    | mount
    v
Docker Volume
    |
    v
Persistent Data
```

---

# 25. Example of Docker Volume

Create a volume:

```bash
docker volume create mydata
```

Run a container:

```bash
docker run -d   --name my-nginx   -v mydata:/data   nginx
```

Now:

```text
Container
   |
   +---- /data
          |
          v
      mydata volume
```

The volume exists independently of the container lifecycle.

---

# 26. Why Use Volumes?

Volumes are useful for:

- Databases
- Application data
- Uploaded files
- Persistent application state
- Shared data between containers

Example:

```text
PostgreSQL Container
        |
        v
 PostgreSQL Volume
        |
        v
   Database Data
```

If the PostgreSQL container is replaced, the volume can remain.

---

# 27. Docker Bind Mount

A bind mount maps a specific directory or file from the host into a container.

Example:

```bash
docker run -v /home/user/app:/app nginx
```

Conceptually:

```text
Host
/home/user/app
       |
       | bind mount
       v
Container
/app
```

Difference:

```text
Volume
------
Managed by Docker


Bind Mount
----------
Uses a specific host path
```

---

# 28. Docker Network

Docker networking allows containers to communicate with:

- Other containers
- The host
- External networks
- The Internet

Example:

```text
Container A
    |
    | Docker Network
    |
Container B
```

---

# 29. Common Docker Network Types

## Bridge

The default network driver on many standard Docker installations.

Example:

```bash
docker network create mynetwork
```

Run containers:

```bash
docker run -d --name app --network mynetwork nginx
```

Containers connected to the same user-defined bridge network can communicate with each other.

---

## Host

With host networking, the container shares the host's network namespace.

```bash
docker run --network host nginx
```

The container does not get a separate virtual network stack in the same way as a bridge-networked container.

---

## None

The container has no normal external network connectivity.

```bash
docker run --network none nginx
```

This can be useful for workloads that should not use networking.

---

## Overlay

Overlay networking is designed for communication across multiple Docker hosts and is associated with Docker Swarm networking.

```text
Host 1                    Host 2
  |                         |
Container A ----------- Container B
          Overlay Network
```

---

# 30. Docker Network Example

Create a network:

```bash
docker network create app-network
```

Run a backend:

```bash
docker run -d   --name backend   --network app-network   my-backend
```

Run a frontend:

```bash
docker run -d   --name frontend   --network app-network   my-frontend
```

The containers can communicate through the Docker network.

For example, the frontend may connect to:

```text
backend:8080
```

---

# 31. Docker Port Mapping

A container port is not automatically published to the host just because the application listens on that port.

Example:

```dockerfile
EXPOSE 8080
```

`EXPOSE` documents the intended container port.

To publish it to the host:

```bash
docker run -p 3000:8080 myapp
```

This means:

```text
Host Port       Container Port
   3000   --->      8080
```

So a request to:

```text
localhost:3000
```

can reach the application listening on:

```text
container:8080
```

---

# 32. EXPOSE vs Port Publishing

### EXPOSE

```dockerfile
EXPOSE 8080
```

Means:

> The application is expected to listen on port 8080 inside the container.

It does not publish the port by itself.

### `-p`

```bash
docker run -p 3000:8080 myapp
```

Means:

> Publish host port 3000 and forward it to container port 8080.

---

# 33. Docker Layers

Docker images are built in layers.

Example:

```dockerfile
FROM ubuntu:22.04

RUN apt-get update

COPY app.py /app/

RUN pip install flask
```

Conceptually:

```text
Layer 4: pip install flask
Layer 3: COPY app.py
Layer 2: apt-get update
Layer 1: ubuntu base image
```

Each Dockerfile instruction can contribute to the image's filesystem/layer structure.

Layering provides benefits such as:

- Image reuse
- Build cache
- Faster rebuilds
- Efficient image distribution

---

# 34. Docker Image Cache

Docker can reuse cached build results when previous build steps have not changed.

For example:

```dockerfile
COPY package.json .
RUN npm install

COPY src ./src
```

If only the source code changes, Docker may reuse the cached dependency installation step.

This is why Dockerfiles are often structured to copy dependency files before application source code.

---

# 35. Docker Registry vs Docker Image

These two concepts are different.

### Image

The actual container image.

```text
myapp:1.0
```

### Registry

The place where the image is stored and distributed.

```text
Docker Hub
ACR
ECR
GHCR
```

Example:

```bash
docker push myregistry/myapp:1.0
```

Flow:

```text
Docker Image
     |
     | docker push
     v
Registry
     |
     | docker pull
     v
Another Docker Host
```

---

# 36. Docker Compose

Docker Compose is used to define and run multi-container applications.

For example:

```text
Frontend
    |
    v
Backend
    |
    v
Database
```

A `compose.yaml` file can define all these services.

Example:

```yaml
services:

  frontend:
    image: my-frontend

  backend:
    image: my-backend

  database:
    image: postgres
```

Start the application:

```bash
docker compose up -d
```

---

# 37. Docker Architecture: Complete Conceptual Flow

A complete Docker workflow can be visualized as:

```text
                 Developer
                     |
                     |
                 Dockerfile
                     |
                     | docker build
                     v
                Docker Image
                     |
             docker push/pull
                     |
                     v
              Container Registry
                     |
                     | docker pull
                     v
                Docker Engine
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Container   Network    Volume
          |
          v
     Application
```

---

# 38. Docker Components Summary

| Component | Purpose |
|---|---|
| Docker Client | Sends commands/API requests to Docker |
| Docker Engine | Manages images, containers, networks and volumes |
| Dockerfile | Defines instructions for building an image |
| Docker Image | Immutable template used to create containers |
| Docker Container | Running/stopped instance created from an image |
| Docker Registry | Stores and distributes images |
| Docker Volume | Provides persistent storage |
| Docker Network | Provides container networking |
| Docker Compose | Defines and runs multi-container applications |
| containerd | Manages container lifecycle underneath Docker Engine |
| runc | Low-level container runtime |
| Namespaces | Provide process/network/filesystem isolation |
| cgroups | Control and limit resource usage |

---

# 39. Docker End-to-End Example

Suppose we have a Python application.

### Step 1: Source Code

```text
app.py
requirements.txt
Dockerfile
```

### Step 2: Dockerfile

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

EXPOSE 8080

CMD ["python", "app.py"]
```

### Step 3: Build

```bash
docker build -t my-python-app:1.0 .
```

Result:

```text
Docker Image
my-python-app:1.0
```

### Step 4: Run

```bash
docker run -d   --name myapp   -p 8080:8080   my-python-app:1.0
```

Result:

```text
Docker Container
myapp
```

### Step 5: Registry

Push the image:

```bash
docker push myregistry/my-python-app:1.0
```

Another server can pull it:

```bash
docker pull myregistry/my-python-app:1.0
```

---

# 40. Docker vs Virtual Machine

A common interview question is:

> What is the difference between a Docker container and a virtual machine?

### Virtual Machine

```text
Physical Server
       |
   Hypervisor
       |
  +----+----+
  |         |
 VM 1      VM 2
 |          |
Guest OS   Guest OS
 |          |
App        App
```

### Docker Container

```text
Physical / Virtual Server
       |
    Host OS
       |
 Docker Engine
       |
  +----+----+
  |    |    |
 C1   C2   C3
 |    |    |
App  App  App
```

### Key Difference

> **A VM normally includes a complete guest operating system, while containers share the host kernel and isolate application processes.**

---

# 41. Why Docker Became Popular

Docker helped standardize application packaging and deployment.

Developers can package:

```text
Application
+
Dependencies
+
Runtime
+
Configuration
```

into an image.

The image can then be used consistently across environments.

This fits very well with:

- DevOps
- CI/CD
- Microservices
- Cloud
- Kubernetes
- Infrastructure automation

---

# 42. Docker in CI/CD

A common CI/CD workflow is:

```text
Developer
    |
    v
Git Repository
    |
    v
CI Pipeline
    |
    | docker build
    v
Docker Image
    |
    | Security Scan
    v
Container Registry
    |
    | docker pull
    v
Kubernetes / VM / Container Platform
```

For example:

```text
GitHub
   |
   v
Jenkins / GitHub Actions / Azure DevOps
   |
   v
Docker Build
   |
   v
ACR / ECR / Docker Hub
   |
   v
AKS / EKS / Kubernetes
```

---

# 43. Docker and Kubernetes

Docker and Kubernetes are related but they are not the same thing.

### Docker

Primarily provides tools for:

- Building images
- Running containers
- Managing containers
- Managing container networks
- Managing container storage

### Kubernetes

Provides orchestration capabilities such as:

- Scheduling containers
- Scaling workloads
- Service discovery
- Rolling deployments
- Self-healing
- Load balancing
- Configuration and secret management

Modern Kubernetes commonly uses container runtimes such as `containerd` rather than requiring the Docker Engine itself on every node.

Conceptually:

```text
Docker / Build Tools
       |
       v
Container Image
       |
       v
Container Registry
       |
       v
Kubernetes
       |
       +---- Pod
       +---- Deployment
       +---- Service
       +---- ConfigMap
       +---- Secret
```

---

# 44. Important Docker Commands

## Check Docker version

```bash
docker version
```

## List images

```bash
docker images
```

## Build an image

```bash
docker build -t myapp:1.0 .
```

## Run a container

```bash
docker run -d --name myapp myapp:1.0
```

## List running containers

```bash
docker ps
```

## List all containers

```bash
docker ps -a
```

## Stop a container

```bash
docker stop myapp
```

## Start a stopped container

```bash
docker start myapp
```

## Remove a container

```bash
docker rm myapp
```

## Remove an image

```bash
docker rmi myapp:1.0
```

## View logs

```bash
docker logs myapp
```

## Execute a command inside a running container

```bash
docker exec -it myapp bash
```

## List networks

```bash
docker network ls
```

## List volumes

```bash
docker volume ls
```

## Pull an image

```bash
docker pull nginx
```

## Push an image

```bash
docker push myregistry/myapp:1.0
```

---

# 45. Important Interview Questions

## Q1. What is Docker?

> Docker is a containerization platform used to build, package, distribute, and run applications as containers.

## Q2. What is containerization?

> Containerization is a method of packaging an application with its dependencies and runtime environment into an isolated container that shares the host operating system kernel.

## Q3. What is virtualization?

> Virtualization creates virtual machines with virtual hardware, and each VM normally runs its own guest operating system.

## Q4. What is a Docker image?

> A Docker image is an immutable, read-only template used to create containers.

## Q5. What is a Docker container?

> A Docker container is an isolated running or stopped instance created from a Docker image.

## Q6. What is a Docker volume?

> A Docker volume provides persistent storage that exists independently of a container's writable layer.

## Q7. What is Docker networking?

> Docker networking provides communication between containers, the host, and external networks.

## Q8. What is Docker Registry?

> A Docker registry stores and distributes container images.

## Q9. What is the difference between Docker image and container?

> An image is the template, while a container is an instance created from that image.

## Q10. What is the difference between Docker and a VM?

> A VM normally includes a complete guest operating system, while containers share the host kernel and isolate application processes.

---

# 46. Final Mental Model

Remember Docker using this simple flow:

```text
                 CODE
                  |
                  v
              Dockerfile
                  |
                  | docker build
                  v
            Docker IMAGE
                  |
                  | docker push
                  v
          CONTAINER REGISTRY
                  |
                  | docker pull
                  v
             DOCKER ENGINE
                  |
          +-------+-------+
          |       |       |
          v       v       v
      Container Network Volume
          |
          v
     APPLICATION
```

And remember the most important concepts:

```text
Virtualization
    ↓
Virtual Machines
    ↓
Each VM normally has its own Guest OS


Containerization
    ↓
Containers
    ↓
Share Host Kernel


Docker
    ↓
Platform for building, packaging,
distributing and running containers


Dockerfile
    ↓
Instructions to build an Image


Image
    ↓
Immutable Template


Container
    ↓
Instance of an Image


Volume
    ↓
Persistent Data


Network
    ↓
Container Communication


Registry
    ↓
Store and Distribute Images
```

# 47. One-Line Definitions for Quick Revision

| Concept | One-Line Definition |
|---|---|
| Virtualization | Creates virtual machines on physical/virtualized hardware |
| Containerization | Packages applications and dependencies into isolated containers |
| Docker | Platform for building, distributing, and running containers |
| Dockerfile | Text file containing image-build instructions |
| Docker Image | Immutable template used to create containers |
| Docker Container | Isolated instance created from an image |
| Docker Engine | Core Docker technology that manages containers and related resources |
| Docker Registry | Repository for storing and distributing images |
| Docker Volume | Persistent storage for container data |
| Docker Network | Provides networking between containers and external systems |
| Docker Compose | Tool for defining and running multi-container applications |
| containerd | Container lifecycle management component |
| runc | Low-level OCI container runtime |
| Namespace | Provides isolation for processes and system resources |
| cgroups | Controls and limits container resource usage |

# 48. Final Takeaway

The complete Docker concept can be summarized as:

> **Containerization packages an application and its dependencies into an isolated container. Docker provides the tools to build that package as an image, store and distribute the image through a registry, and run it as a container. Docker also provides networking, storage, and lifecycle management capabilities around those containers.**

The most important relationship to remember is:

```text
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker run
    ↓
Docker Container
    ↓
Application
```
