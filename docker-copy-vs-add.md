# Docker COPY vs ADD

## 1. Introduction

`COPY` and `ADD` are both Dockerfile instructions used to copy files and directories into a Docker image.

The basic difference is:

> **COPY = Simple and explicit file/directory copying**
>
> **ADD = COPY + additional features such as local archive extraction and remote URL support**

For most Dockerfiles, **COPY is preferred** because it is simpler and more predictable.

---

# 2. COPY

`COPY` copies files or directories from the Docker build context into the image.

### Syntax

```dockerfile
COPY <source> <destination>
```

### Example

Suppose the application directory on the host is:

```text
myapp/
├── Dockerfile
├── index.html
└── app/
    └── app.py
```

Dockerfile:

```dockerfile
FROM ubuntu:22.04

WORKDIR /app

COPY index.html .
COPY app ./app
```

After the build, the image contains:

```text
/app/
├── index.html
└── app/
    └── app.py
```

---

# 3. COPY with WORKDIR

If we use:

```dockerfile
WORKDIR /app

COPY index.html .
```

the destination `.` refers to the current working directory:

```text
/app
```

So the file becomes:

```text
/app/index.html
```

Similarly:

```dockerfile
COPY src ./src
```

copies the local `src` directory into:

```text
/app/src
```

---

# 4. ADD

`ADD` can also copy files and directories into the image.

### Syntax

```dockerfile
ADD <source> <destination>
```

For a simple file copy:

```dockerfile
FROM ubuntu:22.04

WORKDIR /app

ADD index.html .
```

This works similarly to:

```dockerfile
COPY index.html .
```

However, `ADD` has additional behavior that `COPY` does not have.

---

# 5. Main Difference

| Feature | COPY | ADD |
|---|---|---|
| Copy local files | Yes | Yes |
| Copy local directories | Yes | Yes |
| Copy from build context | Yes | Yes |
| Automatically extract local tar archives | No | Yes |
| Remote URL support | No | Historically supported |
| Simpler and predictable | Yes | Less predictable |
| Recommended for normal file copying | Yes | Usually no |

The most important difference is:

```text
COPY
 ↓
Simple file/directory copy
```

```text
ADD
 ↓
File/directory copy
+
Additional features
```

---

# 6. ADD Automatically Extracts Local Tar Archives

One important feature of `ADD` is automatic extraction of local tar archives.

Suppose we have:

```text
myapp/
├── Dockerfile
└── application.tar
```

Dockerfile:

```dockerfile
FROM ubuntu:22.04

WORKDIR /app

ADD application.tar .
```

If `application.tar` contains:

```text
config/
app.py
requirements.txt
```

Docker can automatically extract the archive into:

```text
/app/
├── config/
├── app.py
└── requirements.txt
```

With `COPY`, the archive would simply be copied as a file:

```dockerfile
COPY application.tar .
```

Result:

```text
/app/application.tar
```

It would not automatically extract the archive.

---

# 7. COPY Does Not Extract Archives

For example:

```dockerfile
COPY application.tar /app/
```

The result is:

```text
/app/application.tar
```

If you want to extract it, you would explicitly use a command:

```dockerfile
COPY application.tar /app/

RUN tar -xf /app/application.tar -C /app/
```

This makes the operation explicit.

---

# 8. Remote URLs

Historically, `ADD` has supported URLs such as:

```dockerfile
ADD https://example.com/application.tar.gz /app/
```

However, using `ADD` for remote downloads is generally not the preferred approach.

For downloads, it is usually better to explicitly use tools such as `curl` or `wget` in a `RUN` instruction when appropriate.

For example:

```dockerfile
RUN curl -L https://example.com/application.tar.gz -o /tmp/application.tar.gz
```

This makes the download operation more explicit and gives you better control over the process.

> Note: Docker's current documentation and builder behavior have evolved over time, so for interview purposes the safest rule is: **use COPY for ordinary local files/directories and avoid relying on ADD's extra behavior unless you specifically need it.**

---

# 9. Why COPY Is Preferred

Docker best practices generally favor `COPY` for normal file and directory copying.

For example:

```dockerfile
COPY package.json .
COPY src ./src
```

is clearer than:

```dockerfile
ADD package.json .
ADD src ./src
```

The reason is simple:

> **COPY has one clear purpose: copy files and directories.**

When someone reads the Dockerfile, they immediately know what the instruction is doing.

---

# 10. Practical Example: Node.js Application

Suppose we have:

```text
node-app/
├── Dockerfile
├── package.json
├── package-lock.json
└── src/
    └── server.js
```

A good Dockerfile is:

```dockerfile
FROM node:22

WORKDIR /app

COPY package*.json .

RUN npm install

COPY src ./src

CMD ["node", "src/server.js"]
```

Here we use `COPY` because we only need to copy application files.

There is no reason to use `ADD`.

---

# 11. Practical Example: Java Application

For a Java application:

```text
java-app/
├── Dockerfile
├── pom.xml
└── src/
```

We can write:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package
```

Again, `COPY` is the appropriate choice because we are simply copying:

```text
pom.xml
src/
```

into the build image.

---

# 12. COPY vs ADD in Multi-Stage Builds

`COPY` is also commonly used in multi-stage Dockerfiles.

Example:

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn package

# Runtime stage
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

CMD ["java", "-jar", "app.jar"]
```

Notice:

```dockerfile
COPY --from=builder /app/target/*.jar app.jar
```

This copies the built JAR from the `builder` stage into the runtime stage.

`ADD` is not required here.

---

# 13. Build Context and COPY

When you run:

```bash
docker build -t myapp .
```

the final `.` represents the **build context**.

For example:

```text
myapp/
├── Dockerfile
├── index.html
└── src/
```

Then:

```dockerfile
COPY index.html .
COPY src ./src
```

gets the source files from the build context.

Files outside the build context generally cannot be directly copied with `COPY`.

A `.dockerignore` file can be used to exclude unnecessary files from the build context.

Example:

```text
node_modules
.git
*.log
```

---

# 14. Common Mistake

Do not think:

```dockerfile
COPY /home/rajiv/app/index.html /app/
```

means Docker can always copy any file from the host filesystem.

The source must be within the Docker build context, subject to Docker's build rules.

For example:

```bash
docker build -t myapp .
```

If `.` is:

```text
/home/rajiv/app
```

then:

```dockerfile
COPY index.html /app/
```

can copy:

```text
/home/rajiv/app/index.html
```

into the image.

---

# 15. Easy Way to Remember

Think of `COPY` as a normal photocopy operation:

```text
COPY
 ↓
Take this file/folder
 ↓
Put it here
```

`ADD` is more like:

```text
ADD
 ↓
Copy this file/folder
 +
Possibly perform additional Docker-specific handling
```

Therefore:

> **If you only need to copy files, use COPY.**

---

# 16. Interview Answer

### Question: What is the difference between COPY and ADD in Docker?

A good interview answer is:

> **"`COPY` and `ADD` are both Dockerfile instructions used to add files and directories to an image. `COPY` is designed for straightforward copying from the build context, while `ADD` provides additional features such as automatic extraction of local tar archives. For normal file and directory copying, I prefer `COPY` because it is simpler, more explicit, and predictable."**

### Short Interview Answer

> **"COPY is used for simple and explicit file or directory copying. ADD provides additional capabilities, such as automatic extraction of local tar archives. For normal file copying, COPY is the recommended choice."**

---

# 17. Quick Revision

```text
COPY
 ↓
Local file/directory
 ↓
Destination in image
```

```text
ADD
 ↓
Local file/directory
 +
Additional features
 ↓
Destination in image
```

### Remember

> **COPY = Keep it simple**
>
> **ADD = Copy + extra behavior**

### Best Practice

For most Dockerfiles:

```dockerfile
COPY <source> <destination>
```

Use `ADD` only when you specifically need one of its additional behaviors, such as automatic extraction of a local tar archive.
