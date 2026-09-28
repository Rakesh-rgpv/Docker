# Docker Volume – Beginner-Friendly Guide

## 1. What Is a Docker Volume?

A **Docker Volume** is a storage mechanism used to persist data generated and used by Docker containers.

A volume stores data outside a container's writable layer. This means the data can remain available even when the container is stopped or deleted.

**In simple words:** A container may be temporary, but the data stored in a Docker volume can survive the container's lifecycle.

---

## 2. Why Do We Need Docker Volumes?

By default, files created inside a container are stored in its writable layer. If the container is removed, those files are removed with it.

Docker volumes help us:

- **Persist data:** Keep important data even after a container is deleted.
- **Store database data:** Preserve data for MySQL, PostgreSQL, and other databases.
- **Share data:** Allow multiple containers to access the same volume.
- **Separate data from containers:** Replace or recreate containers without losing persistent data.

---

## 3. Types of Docker Storage

Docker supports three common storage mechanisms:

| Storage Type | Description |
|---|---|
| **Volume** | Storage managed by Docker. |
| **Bind mount** | Maps a specific directory or file on the host into a container. |
| **tmpfs mount** | Stores data in the host's memory; data does not persist after the container or host stops. |

This guide focuses on **Docker volumes**.

---

## 4. How to Create a Docker Volume

Use the `docker volume create` command.

```bash
docker volume create website-data
```

Here, `website-data` is the name of the volume.

### List all volumes

```bash
docker volume ls
```

### Inspect a volume

```bash
docker volume inspect website-data
```

This displays information such as the volume name, driver, and mountpoint.

---

## 5. How to Delete a Docker Volume

To delete a specific volume:

```bash
docker volume rm website-data
```

**Important:** Docker cannot normally remove a volume while it is in use by a container. Stop and remove the container using it first.

### Remove unused volumes

```bash
docker volume prune
```

This removes unused volumes after confirmation. Review the prompt carefully before proceeding.

---

# 6. Practical Example: Hosting a Website with Nginx

In this practical, we will:

1. Create an Nginx container without a volume.
2. Add a sample website inside the container.
3. Delete the container and observe that the website data is lost.
4. Create a Docker volume and attach it to a new Nginx container.
5. Add the sample website to the mounted volume.
6. Delete that container.
7. Create another container using the same volume.
8. Verify that the website data is still available.

> **Prerequisite:** Docker must be installed and running. These commands work with a Docker Engine environment. If you use Docker Desktop, run them in a terminal connected to Docker.

---

## Part A: Demonstrate Data Loss Without a Volume

### Step 1: Create an Nginx container

```bash
docker run -d --name nginx-no-volume -p 8080:80 nginx
```

This command starts an Nginx container and maps host port `8080` to container port `80`.

### Step 2: Create a sample website

Run this command to create an HTML page inside the container:

```bash
docker exec nginx-no-volume sh -c 'echo "<h1>Welcome to My Sample Website</h1><p>This website is running in Nginx.</p>" > /usr/share/nginx/html/index.html'
```

### Step 3: Verify the website

Open the following address in your browser:

```text
http://localhost:8080
```

You should see:

**Welcome to My Sample Website**

### Step 4: Delete the container

```bash
docker rm -f nginx-no-volume
```

The container and its writable layer have been removed.

### Step 5: Create a new container without a volume

```bash
docker run -d --name nginx-no-volume-2 -p 8080:80 nginx
```

Open the website again:

```text
http://localhost:8080
```

The custom website content is no longer there. The new container has its own fresh writable layer and displays the default Nginx page.

**Conclusion:** Data stored only in a container's writable layer does not survive when that container is removed.

Clean up this container before continuing:

```bash
docker rm -f nginx-no-volume-2
```

---

## Part B: Persist Website Data Using a Docker Volume

Now we will store the website files in a Docker volume so that they remain available when containers are replaced.

### Step 1: Create a Docker volume

```bash
docker volume create website-data
```

Verify that it exists:

```bash
docker volume ls
```

### Step 2: Create an Nginx container and mount the volume

```bash
docker run -d \
  --name nginx-volume-1 \
  -p 8080:80 \
  -v website-data:/usr/share/nginx/html \
  nginx
```

### Understand the volume mapping

```text
-v website-data:/usr/share/nginx/html
```

| Part | Meaning |
|---|---|
| `-v` | Specifies a volume or bind mount. |
| `website-data` | Name of the Docker volume. |
| `/usr/share/nginx/html` | Directory inside the container where the volume is mounted. |

Nginx serves website files from `/usr/share/nginx/html`. By mounting the volume at this path, the website files are stored in the volume rather than only in the container's writable layer.

> **Note:** Mounting a volume over a directory hides the image's existing files at that path. For a newly created empty volume, Docker may copy the directory's existing contents into the volume by default. In this practical, we will create our own `index.html` file.

### Step 3: Create the sample website in the mounted directory

```bash
docker exec nginx-volume-1 sh -c 'echo "<h1>Hello from Docker Volume</h1><p>This website data is stored in a persistent volume.</p>" > /usr/share/nginx/html/index.html'
```

The file is written to the mounted volume.

### Step 4: Verify the website

Open:

```text
http://localhost:8080
```

You should see:

**Hello from Docker Volume**

### Step 5: Delete the first container

```bash
docker rm -f nginx-volume-1
```

The container is deleted, but the `website-data` volume remains.

Verify the volume:

```bash
docker volume ls
```

You should still see `website-data`.

### Step 6: Create a new container using the same volume

```bash
docker run -d \
  --name nginx-volume-2 \
  -p 8080:80 \
  -v website-data:/usr/share/nginx/html \
  nginx
```

Notice that we have attached the **same volume**, `website-data`, to the same directory inside the new container.

### Step 7: Verify the website again

Open:

```text
http://localhost:8080
```

You should see the same content:

**Hello from Docker Volume**

The website is still available because its `index.html` file was stored in the volume, not only in the deleted container.

---

## 7. What Happened in This Practical?

| Step | Container | Volume | Result |
|---|---|---|---|
| First example | `nginx-no-volume` | None | Website file was stored in the container's writable layer. |
| Container removed | `nginx-no-volume` deleted | None | Custom website file was lost. |
| Volume example | `nginx-volume-1` | `website-data` | Website file was stored in the mounted volume. |
| Container removed | `nginx-volume-1` deleted | `website-data` remains | Website data is preserved. |
| New container | `nginx-volume-2` | Same `website-data` | Original website content is available again. |

### Architecture

```text
                 Docker Host
        ┌──────────────────────────────┐
        │                              │
        │       Docker Volume          │
        │        website-data          │
        │              │               │
        │              │ Mount         │
        │              ▼               │
        │    ┌────────────────────┐    │
        │    │ Nginx Container 1  │    │
        │    │ /usr/share/nginx/  │    │
        │    │ html               │    │
        │    └────────────────────┘    │
        │              │               │
        │       Container deleted      │
        │              │               │
        │              ▼               │
        │    ┌────────────────────┐    │
        │    │ Nginx Container 2  │    │
        │    │ /usr/share/nginx/  │    │
        │    │ html               │    │
        │    └────────────────────┘    │
        │                              │
        │ Both containers use the      │
        │ same persistent volume.      │
        └──────────────────────────────┘
```

---

## 8. Useful Docker Volume Commands – Quick Reference

| Command | Purpose |
|---|---|
| `docker volume create mydata` | Create a named volume. |
| `docker volume ls` | List volumes. |
| `docker volume inspect mydata` | Display volume details. |
| `docker volume rm mydata` | Remove a volume that is not in use. |
| `docker volume prune` | Remove unused volumes after confirmation. |
| `docker inspect <container-name>` | Inspect container configuration, including mounts. |

---

## 9. Clean Up the Practical

Remove the final container:

```bash
docker rm -f nginx-volume-2
```

If you no longer need the volume, remove it:

```bash
docker volume rm website-data
```

**Warning:** Removing the volume deletes the persistent data stored in it. Do this only after you have finished the practical and no longer need the website files.

---

## 10. Interview Question

### What is a Docker volume, and why do we use it?

**Answer:**

A Docker volume is a Docker-managed storage mechanism used to persist data independently of a container's lifecycle. We use volumes to preserve important data, such as application files and database data, even when a container is deleted and recreated.

### Key Takeaway

**Containers can be replaced; volumes preserve data.** By mounting the same Docker volume into a new container, we can access the data created by a previous container.
