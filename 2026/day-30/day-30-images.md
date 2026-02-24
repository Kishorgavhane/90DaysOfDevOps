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


---

# TASK 4: Working with Running Containers

- Run Nginx in Detached Mode
```bash
docker run -d -p 8080:80 --name mynginx nginx
```

- Open:
```text
http://localhost:8080
```
- View Logs
```bash
docker logs mynginx
```
>container output.

- Real-Time Logs
```bash
docker logs -f mynginx
```

<img width="1086" height="631" alt="image" src="https://github.com/user-attachments/assets/f23463f6-38bd-4429-92ca-b30911ba5a45" />



> Used in production debugging.

- Exec into Container
```bash
docker exec -it mynginx bash
```

- Explore:
```bash
ls /usr/share/nginx/html
```

<img width="1086" height="83" alt="image" src="https://github.com/user-attachments/assets/d2c6a800-d053-4a58-9a8a-5138e53fd7ca" />



- Run Single Command Without Entering
```bash
docker exec mynginx ls /
```


<img width="1086" height="83" alt="image" src="https://github.com/user-attachments/assets/c15f4f88-9631-40f4-b73f-02017cd576f9" />



- Inspect Container
```bash
docker inspect mynginx
```
**output**
```text
[
    {
        "Id": "8e90fc121ab7a9c1df2b610fee7319d47a9673f400a061f56eb76548b2ba4d05",
        "Created": "2026-02-24T06:14:12.858091268Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 2497,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2026-02-24T06:14:12.926084474Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:5cdef4ac3335f68428701c14c5f12992f5e3669ce8ab7309257d263eb7a856b1",
        "ResolvConfPath": "/var/lib/docker/containers/8e90fc121ab7a9c1df2b610fee7319d47a9673f400a061f56eb76548b2ba4d05/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/8e90fc121ab7a9c1df2b610fee7319d47a9673f400a061f56eb76548b2ba4d05/hostname",
        "HostsPath": "/var/lib/docker/containers/8e90fc121ab7a9c1df2b610fee7319d47a9673f400a061f56eb76548b2ba4d05/hosts",
        "LogPath": "/var/lib/docker/containers/8e90fc121ab7a9c1df2b610fee7319d47a9673f400a061f56eb76548b2ba4d05/8e90fc121ab7a9c1df2b610fee7319d47a9673f400a061f56eb76548b2ba4d05-json.log",
        "Name": "/mynginx",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "bridge",
            "PortBindings": {
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                37,
                153
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": [],
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/interrupts",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware",
                "/sys/devices/virtual/powercap"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "ID": "8e90fc121ab7a9c1df2b610fee7319d47a9673f400a061f56eb76548b2ba4d05",
                "LowerDir": "/var/lib/docker/overlay2/fb3477fbed01a18030a24a7cef557fec57f632ffe060aaef88ada7f354a616ac-init/diff:/var/lib/docker/overlay2/2ebae285bb310e3a4a38c875db6e9b4636afdd27949da4a48b3972d9524d2027/diff:/var/lib/docker/overlay2/80dcd8828259090e1b3a89abf83558df76f011605d6e2f4ec9fbd6668fbbf087/diff:/var/lib/docker/overlay2/6c46e742e48ba4f1bbc5069984bbc8ad8755e9a4cc8ffcc0473601032b8c6622/diff:/var/lib/docker/overlay2/f22495c69870339962d2bd9568e5f55015cf5f72f088d7608b4213d21ff824ea/diff:/var/lib/docker/overlay2/ed892a8661aa73e3b2e9e516d6fa589c14fe42c1338a87d52f6516ad94e7ca9b/diff:/var/lib/docker/overlay2/f2176c2249c73c80e9ac8fbb18cd024d69aab796304d0eb2132f5a826bd8facc/diff:/var/lib/docker/overlay2/10d0d5e61ff8e0432d0ac7f903a4ed07fa6c3da3e4139a0312b031d093407471/diff",
                "MergedDir": "/var/lib/docker/overlay2/fb3477fbed01a18030a24a7cef557fec57f632ffe060aaef88ada7f354a616ac/merged",
                "UpperDir": "/var/lib/docker/overlay2/fb3477fbed01a18030a24a7cef557fec57f632ffe060aaef88ada7f354a616ac/diff",
                "WorkDir": "/var/lib/docker/overlay2/fb3477fbed01a18030a24a7cef557fec57f632ffe060aaef88ada7f354a616ac/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "8e90fc121ab7",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.29.5",
                "NJS_VERSION=0.9.5",
                "NJS_RELEASE=1~trixie",
                "ACME_VERSION=0.3.1",
                "PKG_RELEASE=1~trixie",
                "DYNPKG_RELEASE=1~trixie"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "nginx",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "7664e64720d28a4894e9852d5f905b3c792224a0ef4fbc1d3d8cf3fac3446f8e",
            "SandboxKey": "/var/run/docker/netns/7664e64720d2",
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8080"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8080"
                    }
                ]
            },
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "20c7600bbfb55d66678b653d0617931434b23827e131ab7361f33bd4b4aca3cc",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:66:a9:48:b4:4a",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "02:66:a9:48:b4:4a",
                    "DriverOpts": null,
                    "GwPriority": 0,
                    "NetworkID": "5d15801a23abc7c53cd8ed578dde40659e857d8bd2ea5794a6abb95347c3d010",
                    "EndpointID": "20c7600bbfb55d66678b653d0617931434b23827e131ab7361f33bd4b4aca3cc",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": null
                }
            }
        }
    }
]
```

**i Find:**

- IP Address
- Port bindings
- Mounts
- Network mode
- Restart policy

---

# TASK 5: Cleanup

- Stop All Running Containers
```bash
docker stop $(docker ps -q)
```
- Remove All Stopped Containers
```bash
docker container prune
```

<img width="1086" height="345" alt="image" src="https://github.com/user-attachments/assets/c36cdab4-9ee4-4368-83dd-3ea7d287482a" />



- Remove Unused Images
```bash
docker image prune
```
- Check Docker Disk Usage
```bash
docker system df
```

<img width="1086" height="128" alt="image" src="https://github.com/user-attachments/assets/13881894-cb0a-462d-95f1-a7ee81ee6ff2" />



**Shows:**
- Images space
- Containers space
- Volumes space
- Cache size

---

**⭐ Image = Immutable Template
⭐ Container = Runtime Instance
⭐ Layers = Caching + Storage Optimization
⭐ Alpine = Lightweight & Production Friendly
⭐ Lifecycle = Full control over container state
⭐ Logs & Inspect = Debugging tools
⭐ Prune = Resource management**
