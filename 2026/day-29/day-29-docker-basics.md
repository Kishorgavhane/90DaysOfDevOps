# 🚀 DAY 29 – Introduction to Docker

- **Docker**
  
**DDocker is a containerization platform that allows developers to package applications with all dependencies into lightweight containers.**

- **container**
  
**A container is a lightweight isolated environment that runs applications consistently across different systems:**

- Application code
- Runtime
- Libraries
- Dependencies
- System tools

> It runs anywhere — same behavior on:

- laptop
- Testing server
- Production server
- Cloud

## Why Do We Need Containers?

- **Before Docker:**

**“It works on my machine” problem**
- Dependency conflicts
- Different OS environments
- Heavy Virtual Machines

**Docker solves this by:**
- Packaging everything together
- Running isolated environments
- Fast startup
- Lightweight


## Containers vs Virtual Machines

| Feature     | Virtual Machine   | Docker Container      |
| ----------- | ----------------- | --------------------- |
| OS          | Full OS per VM    | Shares host OS        |
| Size        | GBs               | MBs                   |
| Boot time   | Minutes           | Seconds               |
| Performance | Heavy             | Lightweight           |
| Use Case    | Traditional infra | Microservices, DevOps |


**Difference**
- VM = Hardware level virtualization
- Container = OS level virtualization

> Containers use the host kernel.

## Docker Architecture

- Docker has 3 main parts:

### Docker Client

- Command line (docker run, docker ps)
- Talks to daemon

### Docker Daemon

- Background service
- Builds images
- Runs containers

### Docker Registry

- Stores images
- Example: Docker Hub

Flow:
- Client → Daemon → Registry
- Daemon → Pull Image → Create Container

## Practical

**Install Docker**

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

**Check version:**
```bash
docker --version
```

```text
kishor@Kishor:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
17eec7bbc9d7: Pull complete 
ea52d2000f90: Download complete 
Digest: sha256:ef54e839ef541993b4e87f25e752f7cf4238fa55f017957c2eb44077083d7a6a
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

kishor@Kishor:~$ 
```

## What happened internally

- Docker checked local image
- Didn’t find it
- Pulled from Docker Hub
- Created container
- Executed
- Printed message

> That’s your first container!

## Nginx Container

```bash
docker run -d -p 8080:80 --name mynginx nginx
```


<img width="1303" height="187" alt="image" src="https://github.com/user-attachments/assets/c1078b7e-da26-4aef-baa0-ac4e4772334a" />



Open browser:
```text
http://localhost:8080
```

<img width="1303" height="267" alt="image" src="https://github.com/user-attachments/assets/6545528d-5710-4f0b-9f42-5daa97983702" />



## Ubuntu Interactive Mode

```bash
docker run -it ubuntu bash
```

Now inside container.

I Try:
```bash
ls
apt update
```

Exit:
```bash
exit
```

<img width="1303" height="627" alt="image" src="https://github.com/user-attachments/assets/fa96d20e-5f34-4fcd-9850-fd0ee391c571" />


## Docker IMP Commands

| Command                   | Meaning                 |
| ------------------------- | ----------------------- |
| docker ps                 | Running containers      |
| docker ps -a              | All containers          |
| docker stop <id>          | Stop container          |
| docker rm <id>            | Remove container        |
| docker logs <id>          | Check logs              |
| docker exec -it <id> bash | Enter running container |


## Detached vs Interactive Mode

| Mode        | Flag | Meaning            |
| ----------- | ---- | ------------------ |
| Interactive | -it  | Terminal attached  |
| Detached    | -d   | Runs in background |

**Example:**
```bash
docker run -d nginx
```

## Commands

| Command                                                                                                                                                     | Why It Is Used                                                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `docker ps`                                                                                                                                                 | To list all **running containers**                                               |
| `docker ps -a`                                                                                                                                              | To list **all containers** (running + stopped)                                   |
| `docker run -itd ubuntu`                                                                                                                                    | To run Ubuntu container in interactive + detached mode                           |
| `docker run -it ubuntu bash`                                                                                                                                | To run Ubuntu container in interactive mode with bash                            |
| `docker run -d ubuntu`                                                                                                                                      | To run Ubuntu container in background                                            |
| `docker run -d -p 80:80 nginx`                                                                                                                              | To run Nginx container in background and map host port 80 to container port 80   |
| `docker run -itd nginx`                                                                                                                                     | To run Nginx in interactive + detached mode                                      |
| `docker run hello-world`                                                                                                                                    | To verify Docker installation                                                    |
| `docker exec -it <container_id> bash`                                                                                                                       | To enter inside a running container                                              |
| `docker stop <container_id>`                                                                                                                                | To stop a running container                                                      |
| `docker start <container_id>`                                                                                                                               | To start a stopped container                                                     |
| `docker rm <container_id>`                                                                                                                                  | To remove a stopped container                                                    |
| `docker rm -f <container_id>`                                                                                                                               | To force remove a running container                                              |
| `docker --version`                                                                                                                                          | To check Docker version                                                          |
| `docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=testdb -e MYSQL_USER=user1 -e MYSQL_PASSWORD=pass123 -p 3306:3306 mysql:8.0` | To run MySQL container with custom name, environment variables, and port mapping |
| `systemctl status docker`                                                                                                                                   | To check Docker service status                                                   |
| `systemctl status nginx`                                                                                                                                    | To check Nginx service status on host machine                                    |
| `sudo apt-get update`                                                                                                                                       | To update package list                                                           |
| `sudo apt-get install docker.io`                                                                                                                            | To install Docker                                                                |
| `sudo usermod -aG docker ubuntu`                                                                                                                            | To add user to docker group (run Docker without sudo)                            |
| `newgrp docker`                                                                                                                                             | To apply new group permissions without logout                                    |
| `whoami`                                                                                                                                                    | To check current logged-in user                                                  |
| `cat /etc/group`                                                                                                                                            | To check system groups and verify docker group                                   |


---

🔹 **Learned the difference between Virtual Machines and Containers  
🔹 Understood Docker Architecture (Client, Daemon, Registry)  
🔹 Ran my first container (hello-world)  
🔹 Deployed Nginx container and accessed it in browser  
🔹 Explored Ubuntu in interactive mode**  
