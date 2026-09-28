# Docker Multi-Stage Builds

## 1. What is a Dockerfile?

A **Dockerfile** is a text file that contains a set of instructions used by Docker to build a Docker image.

A simple Dockerfile looks like this:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

The basic flow is:

```text
Dockerfile
    |
    | docker build
    v
Docker Image
    |
    | docker run
    v
Container
```

### Common Dockerfile instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Defines the base image |
| `WORKDIR` | Sets the working directory |
| `COPY` | Copies files from the build context into the image |
| `RUN` | Executes a command during image build |
| `EXPOSE` | Documents the port used by the application |
| `CMD` | Defines the default command to run when the container starts |
| `ENTRYPOINT` | Defines the main executable for the container |

---

# 2. What is a Multi-Stage Docker Build?

A **multi-stage Docker build** is a Dockerfile technique in which we use multiple `FROM` instructions to create separate build stages.

A typical application has two different requirements:

1. **Build environment** - tools and dependencies required to compile/build the application.
2. **Runtime environment** - only the software required to run the already-built application.

For example, a Java application may need:

```text
Build Environment:
- JDK
- Maven
- Source code
- Build dependencies
- Maven plugins
```

But after the application is built, the production container may need only:

```text
Runtime Environment:
- JRE
- Application JAR
```

Therefore, we can separate the build environment from the runtime environment.

The basic concept is:

```text
Build Stage
    |
    | Build application
    v
Application Artifact
    |
    | Copy required artifact
    v
Runtime Stage
    |
    v
Final Docker Image
```

---

# 3. Why Do We Need Multi-Stage Builds?

Suppose we build a Java application using a single Docker image:

```text
Docker Image
|
├── JDK
├── Maven
├── Source Code
├── Build Dependencies
├── Maven Plugins
└── Application JAR
```

The application can run, but many components are no longer required at runtime.

Maven and the source code are needed to **build** the application, not necessarily to **run** it.

With a multi-stage build, we can create:

```text
Build Stage
|
├── JDK
├── Maven
├── Source Code
├── Dependencies
└── Application JAR
          |
          | COPY --from
          v
Runtime Stage
|
├── JRE
└── Application JAR
```

The final image contains only the runtime components and the application artifact that we choose to copy.

---

# 4. How Does a Multi-Stage Build Work?

Every `FROM` instruction starts a new build stage.

For example:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder
```

This starts the first stage.

The `FROM` instruction specifies the base image:

```text
maven:3.9-eclipse-temurin-17
```

The `AS builder` part gives the stage a name:

```text
builder
```

The stage name can later be referenced using:

```dockerfile
COPY --from=builder ...
```

The `AS` stage name is optional. If a stage is not named, Docker assigns it a numeric index starting from `0`.

For example:

```dockerfile
COPY --from=0 ...
```

can reference the first stage.

However, naming stages is recommended because it makes the Dockerfile easier to read and maintain.

---

# 5. Complete Multi-Stage Dockerfile Example

The following example builds a Java application using Maven and then creates a separate runtime image.

```dockerfile
# =========================
# Stage 1: Build
# =========================

FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn package


# =========================
# Stage 2: Runtime
# =========================

FROM eclipse-temurin:17-jre

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

---

# 6. Stage 1 - Build Stage

## 6.1 `FROM`

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder
```

This line does two things:

### Selects the base image

```text
maven:3.9-eclipse-temurin-17
```

This image provides an environment containing the required Java/Maven tooling.

### Names the stage

```text
AS builder
```

The stage is given the name:

```text
builder
```

Conceptually:

```text
Stage 1
Name: builder
Base image: Maven + JDK
Purpose: Build the application
```

---

## 6.2 `WORKDIR`

```dockerfile
WORKDIR /app
```

This sets `/app` as the working directory.

If `/app` does not exist, Docker creates it.

Subsequent relative operations use this directory as their working directory.

For example:

```dockerfile
COPY pom.xml .
```

means:

```text
Copy pom.xml to /app/pom.xml
```

---

## 6.3 Copy `pom.xml`

```dockerfile
COPY pom.xml .
```

The `COPY` syntax is:

```text
COPY <source> <destination>
```

Here:

```text
Source:
pom.xml
```

is taken from the Docker build context.

The destination:

```text
.
```

means the current working directory, which is:

```text
/app
```

Therefore:

```text
Host/build context
        |
        | pom.xml
        v
Build stage
        |
        +-- /app/pom.xml
```

### What is `pom.xml`?

`pom.xml` is the Maven Project Object Model file.

It can define:

- Project information
- Dependencies
- Plugins
- Java version
- Build configuration
- Packaging configuration

It is similar in concept to `requirements.txt` in Python because both can define application dependencies, although `pom.xml` provides much broader project/build configuration.

---

# 7. Copy the Source Code

```dockerfile
COPY src ./src
```

This copies the `src` directory from the Docker build context into the `/app/src` directory in the build stage.

For example:

```text
Host
|
└── my-java-app/
    ├── Dockerfile
    ├── pom.xml
    └── src/
        └── main/
            └── java/
                └── Application.java
```

After the `COPY` instructions, the build environment contains:

```text
/app/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── Application.java
```

---

# 8. Build the Application

```dockerfile
RUN mvn package
```

This command executes Maven during the Docker image build.

Maven uses:

```text
pom.xml
+
source code
+
required dependencies
```

to build the application.

The simplified flow is:

```text
pom.xml
    +
src/
    +
Maven dependencies
    |
    | mvn package
    v
Application build
    |
    v
target/
    |
    └── application.jar
```

The generated JAR file is an **application artifact**.

The dependencies required by Maven are installed/downloaded inside the build environment, not into the normal host operating system environment.

Conceptually:

```text
Docker Host
    |
    | docker build
    v
Build Environment
    |
    ├── Linux filesystem
    ├── Java/JDK
    ├── Maven
    ├── Dependencies
    ├── pom.xml
    ├── src/
    └── target/application.jar
```

Modern Docker builds commonly use BuildKit to execute build steps in isolated build environments.

---

# 9. Stage 2 - Runtime Stage

The second `FROM` starts a new stage:

```dockerfile
FROM eclipse-temurin:17-jre
```

This stage is intended for running the application.

Unlike the first stage, it does not need Maven or the JDK used to compile the application.

Conceptually:

```text
Stage 1 - Build
----------------
Maven
JDK
Source Code
Dependencies
    |
    | Build
    v
application.jar
    |
    | Copy artifact
    v
Stage 2 - Runtime
-----------------
JRE
application.jar
```

---

# 10. Runtime `WORKDIR`

```dockerfile
WORKDIR /app
```

The runtime container uses `/app` as its working directory.

---

# 11. Copy the Artifact from the Build Stage

This is the most important line in the multi-stage Dockerfile:

```dockerfile
COPY --from=builder /app/target/*.jar app.jar
```

Break it down:

```text
COPY
 |
 +-- --from=builder
 |       |
 |       +-- Copy from the stage named "builder"
 |
 +-- /app/target/*.jar
 |       |
 |       +-- Source file in the builder stage
 |
 +-- app.jar
         |
         +-- Destination filename in the current stage
```

The flow is:

```text
Stage 1: builder

/app/target/myapp.jar
          |
          | COPY --from=builder
          v
Stage 2: runtime

/app/app.jar
```

Only the artifact specified by the `COPY` instruction is copied into the runtime stage.

The entire build environment is not automatically copied.

---

# 12. `EXPOSE`

```dockerfile
EXPOSE 8080
```

This documents that the application is expected to use port `8080`.

It does not by itself publish the port to the host.

For example, when running the container, you can publish the port with:

```bash
docker run -p 8080:8080 myapp
```

---

# 13. `CMD`

```dockerfile
CMD ["java", "-jar", "app.jar"]
```

This defines the default command that runs when the container starts.

Effectively:

```bash
java -jar app.jar
```

will be executed.

---

# 14. Complete Flow

The complete multi-stage build can be visualized as:

```text
                         Dockerfile
                              |
                              v
              +-----------------------------+
              |       STAGE 1: BUILD        |
              |                             |
              | Maven + JDK                 |
              |                             |
              | pom.xml                     |
              | src/                        |
              |                             |
              | mvn package                 |
              |          |                  |
              |          v                  |
              | target/application.jar      |
              +-------------+---------------+
                            |
                            | COPY --from=builder
                            v
              +-----------------------------+
              |      STAGE 2: RUNTIME       |
              |                             |
              | JRE                         |
              | application.jar             |
              |                             |
              | java -jar app.jar           |
              +-------------+---------------+
                            |
                            v
                     FINAL IMAGE
                            |
                            v
                       CONTAINER
```

The core idea is:

> **Build big, run small.**

---

# 15. Where Are Dependencies Installed During the Build?

This is an important concept.

When Docker executes:

```dockerfile
RUN mvn package
```

Maven needs to download its dependencies.

Those dependencies are downloaded into the build environment created for the build stage.

For example, Maven commonly uses:

```text
/root/.m2/repository/
```

inside the build environment.

They are not automatically installed into the host's normal Maven environment.

Similarly, with Python:

```dockerfile
FROM python:3.12

COPY requirements.txt .

RUN pip install -r requirements.txt
```

the Python packages are installed in the image/build environment rather than being installed into the host's Python environment.

---

# 16. Multi-Stage Build Without Naming Stages

Naming stages is optional.

For example:

```dockerfile
FROM maven:3.9-eclipse-temurin-17

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn package


FROM eclipse-temurin:17-jre

WORKDIR /app

COPY --from=0 /app/target/*.jar app.jar

CMD ["java", "-jar", "app.jar"]
```

The first stage is automatically assigned index `0`.

Therefore:

```dockerfile
COPY --from=0 ...
```

references the first stage.

With a name:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder
```

we can use:

```dockerfile
COPY --from=builder ...
```

Naming is not mandatory, but it is recommended for readability and maintainability.

---

# 17. Multi-Stage Build Benefits

## 17.1 Smaller Final Images

Build tools such as Maven, compilers, and development dependencies do not need to be included in the runtime image.

Instead of:

```text
JDK
Maven
Source Code
Build Dependencies
Application
```

the final image can contain:

```text
JRE
Application
```

This can significantly reduce image size depending on the application.

---

## 17.2 Reduced Attack Surface

The production image can exclude unnecessary tools such as:

```text
Maven
Compiler
Git
Build tools
Source code
```

Fewer unnecessary components can reduce the potential attack surface of the runtime image.

---

## 17.3 Faster Image Transfer

Smaller images generally take less time to:

```text
Push to a registry
Pull from a registry
Transfer between environments
Deploy to Kubernetes
```

This is particularly useful in CI/CD and Kubernetes environments.

---

## 17.4 Cleaner Production Images

The production image contains only the components required to run the application.

This gives a cleaner separation:

```text
Build Environment
-----------------
Tools required to build

Runtime Environment
-------------------
Tools required to run
```

---

## 17.5 Better Separation of Responsibilities

The build stage is responsible for:

```text
Compile
Test
Package
Generate artifact
```

The runtime stage is responsible for:

```text
Run application
```

This separation makes the Dockerfile easier to understand and maintain.

---

# 18. Multi-Stage Build Is Not the Same as Multiple Containers

A multi-stage Dockerfile does not mean that multiple containers will run.

For example:

```text
One Dockerfile
      |
      +-- Stage 1: Build
      |
      +-- Stage 2: Runtime
      |
      v
One final image
      |
      v
Container
```

Multiple containers are a different concept and can be managed using technologies such as Docker Compose or Kubernetes.

---

# 19. Multi-Stage Build Is Not Limited to Java

Multi-stage builds can be used for many technologies.

Examples include:

```text
Java
Node.js
React
Angular
Go
Python
.NET
C/C++
```

A common frontend example is:

```dockerfile
# Build stage
FROM node:22 AS builder

WORKDIR /app

COPY package*.json .
RUN npm install

COPY . .
RUN npm run build


# Runtime stage
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Here:

```text
Node.js + npm
     |
     | npm run build
     v
Static files
     |
     | COPY --from=builder
     v
Nginx
     |
     v
Final image
```

The final image does not need the Node.js build environment.

---

# 20. Multi-Stage Builds in CI/CD

Multi-stage Docker builds are commonly used in CI/CD pipelines.

A typical workflow is:

```text
Developer
    |
    v
Git Repository
    |
    v
CI Pipeline
    |
    v
docker build
    |
    v
Multi-Stage Dockerfile
    |
    +-- Build Stage
    |
    +-- Runtime Stage
    |
    v
Small Production Image
    |
    v
Container Registry
    |
    v
Kubernetes / AKS / EKS
```

For example:

```bash
docker build -t myapp:1.0 .
```

Then:

```bash
docker push myregistry.example.com/myapp:1.0
```

The registry receives the final image produced by the build.

---

# 21. Important Interview Questions

## What is a multi-stage Docker build?

**Answer:**

> A multi-stage Docker build is a Dockerfile technique where multiple `FROM` instructions are used to create separate build stages. One stage can be used to compile or build the application, while another stage is used to run it. We copy only the required artifacts from the build stage to the runtime stage using `COPY --from`. This helps create smaller and cleaner production images.

---

## Why do we use multi-stage builds?

**Answer:**

> We use multi-stage builds to separate the build environment from the runtime environment. This allows us to exclude unnecessary build tools, source code, and development dependencies from the final production image, resulting in a smaller and potentially more secure image.

---

## What does `AS builder` do?

**Answer:**

> `AS builder` assigns a name to a build stage. The stage can later be referenced using `COPY --from=builder`.

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder
```

and:

```dockerfile
COPY --from=builder /app/target/*.jar app.jar
```

---

## Is naming a stage mandatory?

**Answer:**

> No. Stage naming is optional. If a stage is not named, it can be referenced by its numeric index, such as `--from=0`. However, naming stages is recommended for readability and maintainability.

---

## What does `COPY --from=builder` mean?

**Answer:**

> It copies files from the stage named `builder` into the current build stage. It is commonly used to copy the application artifact from a build stage into a minimal runtime stage.

---

## Does the entire first stage go into the final image?

**Answer:**

> No. The runtime stage starts from its own `FROM` image. Only files explicitly copied from the previous stage, such as the application artifact, are included in the final stage.

---

# 22. Key Concepts to Remember

```text
FROM
↓
Starts a new build stage and selects its base image

AS
↓
Optionally gives the build stage a name

WORKDIR
↓
Sets the working directory

COPY
↓
Copies files into the current build stage

RUN
↓
Executes commands during the build

COPY --from
↓
Copies files from another build stage

EXPOSE
↓
Documents the application's port

CMD
↓
Defines the default command used to start the container
```

The most important multi-stage pattern is:

```dockerfile
FROM <build-image> AS builder

# Copy source code
# Install/build dependencies
# Build application
# Generate artifact


FROM <runtime-image>

COPY --from=builder <artifact> <destination>

# Start application
```

## Final Takeaway

The fundamental idea of a multi-stage Docker build is:

> **Use one environment to build the application and a separate, minimal environment to run the application. Copy only the required artifact from the build stage into the runtime stage.**

In simple terms:

```text
BUILD BIG
   ↓
GENERATE ARTIFACT
   ↓
COPY ONLY WHAT IS REQUIRED
   ↓
RUN SMALL
```

This is why multi-stage builds are widely used for production Docker images and modern CI/CD workflows.
