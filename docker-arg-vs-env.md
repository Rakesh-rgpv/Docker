# Docker ARG vs ENV

## 1. Introduction

`ARG` and `ENV` are Dockerfile instructions used to define variables.

Although they look similar, they have an important difference:

> **ARG = Build-time variable**
>
> **ENV = Runtime environment variable**

The easiest way to remember this is:

```text
ARG
 ↓
Used while building the Docker image

ENV
 ↓
Available in the image and running container
```

---

# 2. ARG

`ARG` defines a variable that is available during the **Docker image build process**.

### Basic Syntax

```dockerfile
ARG VARIABLE_NAME=value
```

### Example

```dockerfile
FROM ubuntu:22.04

ARG APP_VERSION=1.0

RUN echo "Building application version $APP_VERSION"
```

Build the image:

```bash
docker build -t myapp .
```

During the build, Docker can use:

```text
APP_VERSION=1.0
```

We can also provide a different value while building:

```bash
docker build --build-arg APP_VERSION=2.0 -t myapp .
```

Now the build uses:

```text
APP_VERSION=2.0
```

---

# 3. Important Point About ARG

`ARG` is primarily a **build-time variable**.

For example:

```dockerfile
FROM ubuntu:22.04

ARG APP_VERSION=1.0

RUN echo "Version: $APP_VERSION"

CMD ["sh"]
```

The `RUN` instruction can use `APP_VERSION` during the build.

However, after the container starts, `APP_VERSION` is **not automatically available as a normal environment variable** inside the container.

For example:

```bash
docker run myapp
```

You should not expect:

```bash
echo $APP_VERSION
```

to automatically return the ARG value.

---

# 4. ENV

`ENV` defines an environment variable that is available in the image and in containers created from that image.

### Basic Syntax

```dockerfile
ENV VARIABLE_NAME=value
```

### Example

```dockerfile
FROM ubuntu:22.04

ENV APP_ENV=production

CMD ["sh", "-c", "echo Environment: $APP_ENV"]
```

Build:

```bash
docker build -t myapp .
```

Run:

```bash
docker run myapp
```

Output:

```text
Environment: production
```

The variable is available when the container is running.

---

# 5. ENV Can Be Overridden at Runtime

One of the important features of `ENV` is that we can override its value when starting the container.

Dockerfile:

```dockerfile
FROM ubuntu:22.04

ENV APP_ENV=production

CMD ["sh", "-c", "echo Environment: $APP_ENV"]
```

Normal execution:

```bash
docker run myapp
```

Output:

```text
Environment: production
```

Override the value:

```bash
docker run -e APP_ENV=development myapp
```

Output:

```text
Environment: development
```

So:

```text
Dockerfile ENV
      ↓
Default runtime value
      ↓
docker run -e
      ↓
Can override the value
```

---

# 6. ARG vs ENV

| Feature | ARG | ENV |
|---|---|---|
| Full meaning | Build Argument | Environment Variable |
| Available during image build | Yes | Yes |
| Available in running container | No, not automatically | Yes |
| Main purpose | Build-time configuration | Runtime configuration |
| Set during build | `--build-arg` | Dockerfile `ENV` |
| Override at container startup | No | Yes, using `-e` |
| Example use | Version/build configuration | Application environment |
| Scope | Build process | Image and container |

---

# 7. Complete Example

Consider this Dockerfile:

```dockerfile
FROM ubuntu:22.04

ARG APP_VERSION=1.0

ENV APP_ENV=production

RUN echo "Building application version: $APP_VERSION"

CMD ["sh", "-c", "echo Application Environment: $APP_ENV"]
```

## Step 1: Build the image

```bash
docker build --build-arg APP_VERSION=2.0 -t myapp .
```

During the build:

```text
ARG APP_VERSION = 2.0
```

The following command executes:

```dockerfile
RUN echo "Building application version: $APP_VERSION"
```

Output during build:

```text
Building application version: 2.0
```

---

## Step 2: Run the container

```bash
docker run myapp
```

The container sees:

```text
Application Environment: production
```

Why?

Because:

```dockerfile
ENV APP_ENV=production
```

creates an environment variable that is available at runtime.

---

## Step 3: Override ENV at Runtime

```bash
docker run -e APP_ENV=development myapp
```

Output:

```text
Application Environment: development
```

The `ENV` value was changed for that container.

---

# 8. Real-World Use Cases

## ARG Use Cases

`ARG` is useful when a value is required **only while building the image**.

Examples:

### Application version

```dockerfile
ARG APP_VERSION=1.0
```

### Base image version

```dockerfile
ARG PYTHON_VERSION=3.12

FROM python:${PYTHON_VERSION}
```

Build:

```bash
docker build --build-arg PYTHON_VERSION=3.13 -t myapp .
```

### Build configuration

```dockerfile
ARG BUILD_ENV=production

RUN echo "Building for $BUILD_ENV"
```

---

# 9. ENV Use Cases

`ENV` is useful when the application needs a value **while the container is running**.

Examples:

### Application environment

```dockerfile
ENV APP_ENV=production
```

### Application port

```dockerfile
ENV APP_PORT=8080
```

### Configuration

```dockerfile
ENV LOG_LEVEL=INFO
```

### Java application example

```dockerfile
ENV SPRING_PROFILES_ACTIVE=production
```

The Java application can read this environment variable when it starts.

---

# 10. ARG and ENV Together

It is also possible to use `ARG` and `ENV` together.

Example:

```dockerfile
FROM ubuntu:22.04

ARG APP_VERSION=1.0

ENV APP_VERSION=$APP_VERSION

CMD ["sh", "-c", "echo Application version: $APP_VERSION"]
```

Build:

```bash
docker build --build-arg APP_VERSION=2.0 -t myapp .
```

Here:

```text
ARG APP_VERSION
      ↓
Gets value during build
      ↓
ENV APP_VERSION=$APP_VERSION
      ↓
Value becomes an environment variable
      ↓
Available in the running container
```

Run:

```bash
docker run myapp
```

Output:

```text
Application version: 2.0
```

This is a useful pattern when a value needs to be supplied during the build and also made available at runtime.

---

# 11. ARG Before FROM

A special feature of `ARG` is that an `ARG` can be declared before `FROM` and used to select the base image version.

Example:

```dockerfile
ARG PYTHON_VERSION=3.12

FROM python:${PYTHON_VERSION}

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt
```

Build:

```bash
docker build --build-arg PYTHON_VERSION=3.13 -t myapp .
```

Docker will use:

```text
python:3.13
```

as the base image.

This is a common use of `ARG`.

---

# 12. Important: ARG Is Not a Secret Mechanism

Do **not** use `ARG` to safely pass passwords, API keys, tokens, or other sensitive information.

For example, avoid:

```dockerfile
ARG DB_PASSWORD=MyPassword123
```

Build arguments can potentially be exposed through image history or build metadata depending on how they are used.

For secrets, use Docker/BuildKit secret mechanisms or your CI/CD platform's secret management.

---

# 13. ARG vs ENV: Simple Analogy

Think about building a house.

### ARG

`ARG` is like a value needed by the construction team while building the house.

```text
Construction phase
       ↓
ARG
       ↓
Used during build
```

### ENV

`ENV` is like a setting that the people living in the house use after the house is built.

```text
House is built
       ↓
ENV
       ↓
Used by the running application
```

---

# 14. Easy Diagram

```text
                Dockerfile
                    |
          +---------+---------+
          |                   |
         ARG                 ENV
          |                   |
          ↓                   ↓
    Build Time            Image/Runtime
          |                   |
          ↓                   ↓
   docker build          docker run
          |                   |
          |              -e VARIABLE=value
          ↓                   ↓
    Build process       Running container
```

---

# 15. Interview Answer

### Question: What is the difference between ARG and ENV in Docker?

A good interview answer is:

> **"`ARG` is used for build-time variables and is available during the Docker image build process. `ENV` is used to define environment variables that are available in the image and in the running container. We can pass an ARG value using `--build-arg`, while an ENV variable can be overridden at runtime using `docker run -e`."**

### Short Interview Answer

> **"ARG is mainly for build-time configuration, whereas ENV is for runtime configuration."**

---

# 16. Quick Revision

```text
ARG
 ↓
Build-time
 ↓
docker build --build-arg
```

```text
ENV
 ↓
Runtime
 ↓
docker run -e
```

### Remember

> **ARG = Build**
>
> **ENV = Runtime**

And one important security rule:

> **Do not use ARG or ENV for sensitive secrets unless you understand the exposure implications.**
