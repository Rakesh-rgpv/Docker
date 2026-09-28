# Docker CMD vs ENTRYPOINT

## 1. Introduction

`CMD` and `ENTRYPOINT` are Dockerfile instructions used to define what command should run when a container starts.

The main difference is:

- **CMD** provides the default command or default arguments.
- **ENTRYPOINT** defines the main executable/command of the container.

A simple way to remember:

> **CMD = Default behavior**
>
> **ENTRYPOINT = Main application**

---

## 2. CMD

`CMD` specifies the default command that Docker runs when the container starts.

### Example

```dockerfile
FROM ubuntu:22.04

CMD ["echo", "Hello from Docker"]
```

Build the image:

```bash
docker build -t cmd-demo .
```

Run the container:

```bash
docker run cmd-demo
```

Output:

```text
Hello from Docker
```

### CMD can be overridden

For example:

```bash
docker run cmd-demo echo "Hello Rajiv"
```

Output:

```text
Hello Rajiv
```

The command provided after the image name replaces the `CMD`.

So:

```dockerfile
CMD ["echo", "Hello from Docker"]
```

is a **default command**, not a fixed command.

---

## 3. ENTRYPOINT

`ENTRYPOINT` defines the main executable of the container.

### Example

```dockerfile
FROM ubuntu:22.04

ENTRYPOINT ["echo"]
```

Build:

```bash
docker build -t entrypoint-demo .
```

Run:

```bash
docker run entrypoint-demo "Hello from Docker"
```

Output:

```text
Hello from Docker
```

Here:

```text
ENTRYPOINT = echo
Argument    = Hello from Docker
```

Docker effectively runs:

```bash
echo "Hello from Docker"
```

Unlike `CMD`, the argument supplied after the image name is normally passed to the `ENTRYPOINT`.

---

## 4. CMD vs ENTRYPOINT

| Feature | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Provides default command/arguments | Defines the main executable |
| Can be overridden at runtime? | Yes, easily | Not normally; runtime arguments are appended |
| Typical use | Default behavior | Main application |
| Runtime arguments | Replace CMD | Usually passed as arguments |
| Best mental model | Default | Fixed/main executable |

---

## 5. CMD Example

Dockerfile:

```dockerfile
FROM ubuntu:22.04

CMD ["echo", "Hello Docker"]
```

Run:

```bash
docker run myimage
```

Docker executes:

```bash
echo "Hello Docker"
```

If we run:

```bash
docker run myimage echo "Hello Rajiv"
```

Docker executes:

```bash
echo "Hello Rajiv"
```

The original `CMD` is replaced.

---

## 6. ENTRYPOINT Example

Dockerfile:

```dockerfile
FROM ubuntu:22.04

ENTRYPOINT ["echo"]
```

Run:

```bash
docker run myimage "Hello Docker"
```

Docker executes:

```bash
echo "Hello Docker"
```

If we run:

```bash
docker run myimage "Hello Rajiv"
```

Docker executes:

```bash
echo "Hello Rajiv"
```

The `echo` executable remains the entrypoint, while the runtime value becomes its argument.

---

## 7. Using CMD and ENTRYPOINT Together

This is one of the most important patterns.

```dockerfile
FROM ubuntu:22.04

ENTRYPOINT ["echo"]
CMD ["Hello Docker"]
```

Run:

```bash
docker run myimage
```

Docker executes:

```bash
echo "Hello Docker"
```

If we run:

```bash
docker run myimage "Hello Rajiv"
```

Docker executes:

```bash
echo "Hello Rajiv"
```

In this pattern:

- `ENTRYPOINT` = executable
- `CMD` = default argument

So:

```text
ENTRYPOINT + CMD
     ↓
echo + "Hello Docker"
```

---

## 8. Real-World Application Example

Suppose we have a Java application.

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY app.jar .

ENTRYPOINT ["java", "-jar", "app.jar"]
```

When the container starts:

```bash
docker run my-java-app
```

Docker runs:

```bash
java -jar app.jar
```

Here `ENTRYPOINT` ensures that the Java application is the main process of the container.

---

## 9. ENTRYPOINT with CMD for Java

A more flexible example:

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY app.jar .

ENTRYPOINT ["java", "-jar", "app.jar"]

CMD ["--server.port=8080"]
```

Normal execution:

```bash
docker run my-java-app
```

Results in:

```bash
java -jar app.jar --server.port=8080
```

If we provide another argument:

```bash
docker run my-java-app --server.port=9090
```

Results in:

```bash
java -jar app.jar --server.port=9090
```

The `ENTRYPOINT` remains the same, while the `CMD` default argument is replaced.

---

## 10. Shell Form vs Exec Form

Both `CMD` and `ENTRYPOINT` support two common forms.

### Exec Form

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

This is generally preferred because Docker can run the application directly as the container's main process.

### Shell Form

```dockerfile
CMD nginx -g "daemon off;"
```

```dockerfile
ENTRYPOINT java -jar app.jar
```

Shell form involves a shell and can behave differently with signal handling and argument processing.

For most application containers, prefer the **exec form**.

---

## 11. Easy Way to Remember

Think about a calculator application.

### ENTRYPOINT

```dockerfile
ENTRYPOINT ["python", "calculator.py"]
```

This defines the main application.

### CMD

```dockerfile
CMD ["10", "20"]
```

These are the default arguments.

So:

```bash
docker run calculator
```

effectively runs:

```bash
python calculator.py 10 20
```

If we run:

```bash
docker run calculator 50 100
```

the result is effectively:

```bash
python calculator.py 50 100
```

The main application remains the same.

---

## 12. Interview Answer

### Question: What is the difference between CMD and ENTRYPOINT?

A good interview answer is:

> **“CMD and ENTRYPOINT both define what runs when a Docker container starts. CMD provides a default command or default arguments and can be easily overridden at runtime. ENTRYPOINT defines the main executable of the container, and runtime arguments are normally passed to it. A common pattern is to use ENTRYPOINT for the main application and CMD for its default arguments.”**

### Short version

> **“CMD is a default that can be replaced, whereas ENTRYPOINT defines the main executable. When used together, ENTRYPOINT defines the application and CMD provides its default arguments.”**

---

## 13. Final Summary

```text
CMD
 ↓
Default command / default arguments
 ↓
Can be replaced at runtime
```

```text
ENTRYPOINT
 ↓
Main executable
 ↓
Runtime arguments are passed to it
```

Together:

```text
ENTRYPOINT + CMD
       ↓
Main application + Default arguments
```

### Key Interview Rule

> **ENTRYPOINT = What should always run**
>
> **CMD = What should run by default / default arguments**
