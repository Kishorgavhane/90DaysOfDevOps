# DAY 30 – Docker Images & Container Lifecycle


## Docker Image

> A Docker Image is a read-only template used to create containers.

**Think like this:**
- Image = Blueprint
- Container = Running house built from blueprint

## Relationship Between Image & Container

| Image     | Container            |
| --------- | -------------------- |
| Static    | Running instance     |
| Read-only | Writable layer added |
| Template  | Live application     |


- **One image → Many containers**

### Example:

```bash
docker run nginx
docker run nginx
docker run nginx
```
- All 3 containers come from same image.

---

# TASK 1: Docker Images

**Pull Images**
```bash
docker pull nginx
docker pull ubuntu
docker pull alpine
```
- **internally happened**

- Docker contacts Docker Hub
- Downloads image layers
- Stores them locally in `/var/lib/docker`
- Images are cached for reuse

<img width="1102" height="328" alt="image" src="https://github.com/user-attachments/assets/4d5dd076-8f65-48e5-ae35-9837511447e0" />


**Check:**
- docker images

<img width="1086" height="136" alt="image" src="https://github.com/user-attachments/assets/d01ac368-1c7b-4bee-a575-f74773b58e19" />


## Compare Ubuntu vs Alpine

**My Observation:**

- Ubuntu → ~70.1MB
- Alpine → ~8.44MB

**Why Alpine is Smaller?**

| Ubuntu                | Alpine         |
| --------------------- | -------------- |
| Full Linux distro     | Minimal Linux  |
| Uses glibc            | Uses musl      |
| More packages         | Only essential |
| Larger attack surface | More secure    |



# Docker Nginx Image Inspection

## Command Used

```bash
docker inspect nginx
```
**Output**

```text
[
    {
        "Id": "sha256:5cdef4ac3335f68428701c14c5f12992f5e3669ce8ab7309257d263eb7a856b1",
        "RepoTags": [
            "nginx:latest"
        ],
        "RepoDigests": [
            "nginx@sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208"
        ],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2026-02-04T23:53:09.258787756Z",
        "DockerVersion": "",
        "Author": "",
        "Config": {
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.29.5",
                "NJS_VERSION=0.9.5",
                "NJS_RELEASE=1~trixie",
                "ACME_VERSION=0.3.1",
                "PKG_RELEASE=1~trixie",
                "DYNPKG_RELEASE=1~trixie"
            ],
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 160850673,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/80dcd8828259090e1b3a89abf83558df76f011605d6e2f4ec9fbd6668fbbf087/diff:/var/lib/docker/overlay2/6c46e742e48ba4f1bbc5069984bbc8ad8755e9a4cc8ffcc0473601032b8c6622/diff:/var/lib/docker/overlay2/f22495c69870339962d2bd9568e5f55015cf5f72f088d7608b4213d21ff824ea/diff:/var/lib/docker/overlay2/ed892a8661aa73e3b2e9e516d6fa589c14fe42c1338a87d52f6516ad94e7ca9b/diff:/var/lib/docker/overlay2/f2176c2249c73c80e9ac8fbb18cd024d69aab796304d0eb2132f5a826bd8facc/diff:/var/lib/docker/overlay2/10d0d5e61ff8e0432d0ac7f903a4ed07fa6c3da3e4139a0312b031d093407471/diff",
                "MergedDir": "/var/lib/docker/overlay2/2ebae285bb310e3a4a38c875db6e9b4636afdd27949da4a48b3972d9524d2027/merged",
                "UpperDir": "/var/lib/docker/overlay2/2ebae285bb310e3a4a38c875db6e9b4636afdd27949da4a48b3972d9524d2027/diff",
                "WorkDir": "/var/lib/docker/overlay2/2ebae285bb310e3a4a38c875db6e9b4636afdd27949da4a48b3972d9524d2027/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:a8ff6f8cbdfd6741c10dd183560df7212db666db046768b0f05bbc3904515f03",
                "sha256:edcb98f6af683f89a724d9da7bf8927059c91c86db4723a42201cd227340d7b5",
                "sha256:18b09c39ca9f595897956456d144ca812ba219cfe72cee888945b7050fc53b38",
                "sha256:bced0e9a39b0302b03e79b279a5de8394544197578327bfe3108a989b4a7154e",
                "sha256:b9390f60b84fa6b5e7772d3c32dd1e141eb12f34e4cd98dd03da87e7552d76fe",
                "sha256:627c009aa11539cb60bc61ce3709ab81059b224674abd8e06f27b26798969155",
                "sha256:4952de04fe7e4a2b63ed8ac879f7bb23cefa98d6005677c59ebd01fe27d02ba2"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

**See:**

- Image ID
- OS type
- Architecture
- Exposed ports
- Environment variables
- Layer hashes
- **Inspect is used in debugging and production troubleshooting.**

**image remove no need**
```bash
docker rmi <ID/NAME>
```

<img width="1086" height="523" alt="image" src="https://github.com/user-attachments/assets/549d369b-2924-4162-8de1-3c0995278e36" />


| Work         | Command                   |
| ------------ | ------------------------- |
| Single image | `docker rmi <id>`         |
| Name se      | `docker rmi nginx:latest` |
| Force        | `docker rmi -f`           |
| Unused clean | `docker image prune`      |
| Full clean   | `docker system prune -a`  |


---

# TASK 2: Image Layers

**Run:**
```bash
docker image history nginx
```

<img width="1086" height="363" alt="image" src="https://github.com/user-attachments/assets/69a7cb73-b449-4650-9f4f-56f68d1fcd50" />


- I will see multiple entries.

**Each line = One Layer**

- **When building an image:**

Every instruction in Dockerfile creates a new layer.

**Example:**
```bash
FROM ubuntu
RUN apt update
RUN apt install nginx
COPY app /app
```
- This creates 4 layers.

## Why Docker Uses Layers?

- Faster rebuild (caching)
- Space optimization
- Efficient version control
- Shared base images

**Example:**

> If 10 images use ubuntu base → stored only once.

---

## TASK 3: Container Lifecycle

- **Create (Without Starting)**

```bash
docker create nginx
```

> State: *Created*

Check:
```bash
docker ps -a
```

- **Start**

```bash
docker start 3e3038c76782
```
> State: *Running*

- **Pause**

```bash
docker pause 3e3038c76782
```
> state: *Paused*
> Container process is frozen.

- **Unpause**

```bash
docker unpause 3e3038c76782
```
> state: start

- **Stop**
```bash
docker stop 3e3038c76782
```
> Graceful shutdown (SIGTERM).

- **Restart**
```bash
docker restart 3e3038c76782
```

> Stop + Start.

- **Kill**
```bash
docker kill 3e3038c76782
```

> Immediate stop (SIGKILL).

- **Remove**
```bash
docker rm 3e3038c76782
```

> Container deleted permanently.

<img width="1086" height="631" alt="image" src="https://github.com/user-attachments/assets/c252970e-517b-4066-8f91-3bf0e2ed5753" />


