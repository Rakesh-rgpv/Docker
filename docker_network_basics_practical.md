# Docker Networking: Beginner-to-Practical Guide

## 1. What Is a Docker Network?

A **Docker network** allows containers to communicate with one another,
with the Docker host, and---depending on the network
configuration---with external networks such as the internet.

For example, a web application may have: - A **frontend** container that
serves the user interface. - A **backend** container that processes
requests.

Docker networking determines how these containers can reach one another.

``` text
User → Frontend Container → Backend Container
```

Containers attached to the same user-defined bridge network can usually
communicate using their container names.

------------------------------------------------------------------------

## 2. Docker's Default Networks

Check the networks available on your Docker host:

``` bash
docker network ls
```

Docker commonly creates these built-in networks:

  -----------------------------------------------------------------------
  Network                 Driver                  Meaning
  ----------------------- ----------------------- -----------------------
  `bridge`                `bridge`                Default bridge network
                                                  for containers when no
                                                  network is specified.

  `host`                  `host`                  Container shares the
                                                  host's network stack;
                                                  it does not get a
                                                  separate container IP.

  `none`                  `null`                  Container has no
                                                  external network
                                                  connectivity, apart
                                                  from its loopback
                                                  interface.
  -----------------------------------------------------------------------

> **Note:** The built-in network is named `none`, not `null`. `null` is
> the driver name shown for that network.

### 2.1 Bridge Network

The default `bridge` network gives containers private IP addresses on a
Docker-managed bridge network.

``` bash
docker run -dit --name bridge-demo alpine sh
docker inspect bridge-demo
```

Containers on the default bridge can communicate by IP address, but
automatic container-name DNS resolution is generally not provided there.
For name-based communication, use a user-defined bridge network.

### 2.2 Host Network

The host network shares the host's network stack. The container does not
receive a separate IP address.

``` bash
docker run -d --name host-demo --network host nginx
```

Port publishing (`-p`) is not needed and is ignored in host mode.

### 2.3 None Network

The `none` network isolates the container from external networking.

``` bash
docker run -dit --name none-demo --network none alpine sh
```

The container can use its loopback interface, but it cannot normally
communicate with other containers or the internet.

------------------------------------------------------------------------

## 3. Create Your Own Docker Networks

We will create two separate networks: - `frontend-net` for the frontend
container. - `backend-net` for the backend container.

``` bash
docker network create frontend-net
docker network create backend-net
```

Verify:

``` bash
docker network ls
```

Inspect a network:

``` bash
docker network inspect frontend-net
docker network inspect backend-net
```

------------------------------------------------------------------------

## 4. Create One Container on Each Network

We will use lightweight Alpine Linux containers. The backend will run a
tiny HTTP server on port `8080`.

### 4.1 Create the Frontend Container

``` bash
docker run -dit --name frontend --network frontend-net alpine sh
```

### 4.2 Create the Backend Container

``` bash
docker run -dit --name backend --network backend-net alpine sh -c "echo 'Hello from the backend container!' > /tmp/index.html && httpd -f -p 8080 -h /tmp"
```

Check that both containers are running:

``` bash
docker ps
```

At this point: - `frontend` is attached to `frontend-net`. - `backend`
is attached to `backend-net`. - The containers are on separate networks.

------------------------------------------------------------------------

## 5. Try Communication Before Connecting the Networks

Run this command from the frontend container:

``` bash
docker exec frontend wget -T 3 -qO- http://backend:8080
```

**Expected result:** The request fails because the frontend and backend
are on separate networks. The frontend cannot resolve the backend's name
through its current network.

This is the isolation we want to demonstrate.

------------------------------------------------------------------------

## 6. Connect the Frontend Container to the Backend Network

Docker lets a running container connect to an additional network with
`docker network connect`.

``` bash
docker network connect backend-net frontend
```

Now the frontend is attached to both networks:

``` text
                    frontend-net
                         |
                    [frontend]
                         |
                    backend-net
                         |
                    [backend]
```

The backend remains on `backend-net`, and the frontend is now connected
to both `frontend-net` and `backend-net`.

Verify the network attachments:

``` bash
docker inspect frontend
docker network inspect backend-net
```

------------------------------------------------------------------------

## 7. Test Communication Again

Run the same request from the frontend container:

``` bash
docker exec frontend wget -T 3 -qO- http://backend:8080
```

**Expected output:**

``` text
Hello from the backend container!
```

It works because both containers now share `backend-net`, and Docker's
embedded DNS can resolve the backend container name on that user-defined
network.

------------------------------------------------------------------------

## 8. What Did We Achieve?

1.  Created two separate networks: `frontend-net` and `backend-net`.
2.  Created the `frontend` container on `frontend-net`.
3.  Created the `backend` container on `backend-net`.
4.  Tried to access the backend from the frontend; it failed because the
    containers were on separate networks.
5.  Connected the frontend to `backend-net` using
    `docker network connect`.
6.  Retried the request; it succeeded.

**Key takeaway:** Containers on separate networks are isolated from one
another by default. Connecting a container to a shared user-defined
network enables network communication between them, subject to service
availability and firewall rules.

------------------------------------------------------------------------

## 9. Cleanup (Optional)

After completing the exercise, remove the containers and networks:

``` bash
docker rm -f frontend backend
docker network rm frontend-net backend-net
```
