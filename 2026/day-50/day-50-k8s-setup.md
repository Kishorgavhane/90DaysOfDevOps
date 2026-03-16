# Day 50 – Kubernetes Architecture and Cluster Setup

> **#90DaysOfDevOps | DevOps Ka Josh | TrainWithShubham**

---

## My Journey on Day 50


Today I set up my first ever Kubernetes cluster, broke it on purpose, fixed it, and explored what's actually running inside. Here's everything I did, everything I learned, and every mistake I made along the way.

---

## Task 1: The Kubernetes Story (From My Memory)

Before touching the terminal, I wrote down what I remembered.

**Why was Kubernetes created?**
Docker solves "how do I run one container on one machine." But in production, I need answers to harder questions — which server should this container run on? What if that server crashes at 3am? How do I roll out a new version without taking the app down? How do I handle a traffic spike? Docker has no answers. Kubernetes was built to answer all of it.

**Who created it and what inspired it?**
Google built and open-sourced Kubernetes in 2014. But the real origin story is older — Google had an internal system called **Borg** that had been orchestrating their containers at massive scale for over a decade. The Kubernetes team took the lessons from Borg and built a version the whole world could use. It is now maintained by the **Cloud Native Computing Foundation (CNCF)**.

**What does the name mean?**
"Kubernetes" is a Greek word meaning **helmsman** — the person who steers a ship. That's exactly what it does — it steers containers. The short form **K8s** replaces the 8 middle letters with the number 8.

---

## Task 2: Kubernetes Architecture

### The Big Picture

Kubernetes has two sides — the **Control Plane** (the brain) and the **Worker Nodes** (the muscle). The Control Plane decides what should happen. The Worker Nodes make it happen.

```
┌──────────────────────────────────────────────────────────────┐
│                       CONTROL PLANE                          │
│                                                              │
│   ┌──────────────────────────────────────────────────────┐   │
│   │                    API Server                        │   │
│   │      Every single command goes through this          │   │
│   └──────────────────────────────────────────────────────┘   │
│          │               │                  │                │
│   ┌──────▼─────┐  ┌──────▼─────┐  ┌─────────▼──────────┐    │
│   │    etcd    │  │ Scheduler  │  │ Controller Manager  │    │
│   │ cluster DB │  │ node picker│  │  reconciles state   │    │
│   └────────────┘  └────────────┘  └────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
                           │
           ┌───────────────▼──────────────────┐
           │           WORKER NODE            │
           │                                  │
           │  ┌────────────────────────────┐  │
           │  │          kubelet           │  │
           │  │  node agent — manages pods │  │
           │  └────────────────────────────┘  │
           │  ┌─────────────┐ ┌────────────┐  │
           │  │ kube-proxy  │ │ Container  │  │
           │  │ networking  │ │  Runtime   │  │
           │  └─────────────┘ └────────────┘  │
           │  ┌──────────┐  ┌──────────┐      │
           │  │  Pod A   │  │  Pod B   │      │
           │  └──────────┘  └──────────┘      │
           └──────────────────────────────────┘
```

### Component Breakdown

**Control Plane:**

| Component | What it actually does |
|---|---|
| **API Server** | The single front door to the entire cluster. Every `kubectl` command, every internal component — everything talks to this. Nothing bypasses it. |
| **etcd** | A key-value database storing the entire state of the cluster. What pods exist, which nodes are healthy, what deployments are running — all of it lives here. |
| **Scheduler** | When a new Pod needs to run, the Scheduler looks at all available nodes and picks the best one based on available CPU, memory, and other constraints. |
| **Controller Manager** | Runs in a loop constantly watching the cluster. If desired state says "3 replicas" but only 2 are running, the Controller Manager notices and creates the missing one. |

**Worker Node:**

| Component | What it actually does |
|---|---|
| **kubelet** | An agent running on every node. Talks to the API Server, receives instructions, and tells the Container Runtime to start or stop containers. |
| **kube-proxy** | Manages networking rules on each node so pods can communicate with each other and the outside world. |
| **Container Runtime** | The engine that actually runs containers. Kubernetes uses `containerd` or `CRI-O` — not Docker directly anymore. |

### Understanding the Flow: What WILL happen when I run `kubectl apply -f pod.yaml`?

> ⚠️ **Note:** I have not run this command yet. This is purely theoretical understanding of how the architecture works internally. The practical implementation of `kubectl apply` will happen in **Day 51** when I work with Pods and Deployments.

Here is how the request flows through each component:

1. `kubectl` sends the request to the **API Server**
2. API Server **authenticates** the request and **validates** the YAML
3. The desired state gets **written to etcd**
4. The **Scheduler** notices a new Pod exists with no node assigned — picks the best node
5. The **kubelet** on that node sees the assignment and tells **containerd** to pull the image and start the container
6. The **Controller Manager** watches continuously — if the Pod crashes, it creates a replacement

### Failure Scenarios

**What if the API Server goes down?**
Control is lost — `kubectl` stops working, no new scheduling happens. But existing running pods keep running. The data plane survives without the control plane.

**What if a Worker Node goes down?**
The Controller Manager detects `NotReady`. It reschedules all pods from that dead node onto other healthy nodes automatically. This is Kubernetes self-healing.

---

## Task 3: Install kubectl

`kubectl` is the command line tool I use to talk to my Kubernetes cluster. Think of it as the remote control for the cluster. Without it, there is no way to issue commands.

```bash
# Download the latest kubectl binary for Linux (amd64)
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
  https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make the binary executable
chmod +x kubectl

# Move it to /usr/local/bin so I can run it from anywhere in terminal
sudo mv kubectl /usr/local/bin/

# Verify it installed correctly
kubectl version --client
```

### 📸 Screenshot — kubectl installed successfully

![kubectl install](screenshots/kubectl-install.png)

**Output I got:**
```
Client Version: v1.35.2
Kustomize Version: v5.7.1
```

This confirms kubectl is installed and working. It only shows the client version here — that is expected because no cluster exists yet at this point. The server version appears only after connecting to a cluster.

---

## Task 4: Set Up the Local Cluster with kind

### Why I chose kind over minikube

| | kind | minikube |
|---|---|---|
| Runs via | Docker containers | Docker or VM |
| Speed | Very fast | Slightly slower |
| RAM usage | Lower | Higher |
| Best for | Quick testing, CI pipelines | Richer local dev features |

I chose **kind** because Docker was already installed on my machine and kind is lightweight, fast, and perfect for getting started.

### Commands I ran

```bash
# Download the kind binary
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64

# Make it executable
chmod +x ./kind

# Move to PATH so it is accessible from anywhere
sudo mv ./kind /usr/local/bin/kind

# Create my first Kubernetes cluster
kind create cluster --name devops-cluster

# Confirm the cluster is alive
kubectl cluster-info
kubectl get nodes
```

### 📸 Screenshot — cluster created and verified

![kind cluster created](screenshots/kind-cluster-create.png)

**What happened line by line during `kind create cluster`:**

```
Creating cluster "devops-cluster" ...
✓ Ensuring node image (kindest/node:v1.35.1)  ← pulled a Docker image that acts as a K8s node
✓ Preparing nodes                              ← started that Docker container
✓ Writing configuration                        ← created ~/.kube/config automatically
✓ Starting control-plane                       ← API server, etcd, scheduler all started inside
✓ Installing CNI                               ← set up pod-to-pod networking (kindnet plugin)
✓ Installing StorageClass                      ← set up local storage provisioner
Set kubectl context to "kind-devops-cluster"   ← kubectl now knows to talk to this cluster
```

After this, `kubectl cluster-info` confirmed:
- API Server is alive at `https://127.0.0.1:39519`
- CoreDNS is running inside the cluster for internal DNS

And `kubectl get nodes` showed:
```
NAME                          STATUS   ROLES          AGE   VERSION
devops-cluster-control-plane  Ready    control-plane  41s   v1.35.1
```
`STATUS: Ready` — my cluster is up, healthy, and ready for workloads.

---

## Task 5: Explore the Cluster

### 📸 Screenshot — exploring namespaces and pods

![cluster exploration](screenshots/cluster-explore.png)

### `kubectl get nodes`

```bash
kubectl get nodes
```
**What this command does:** Lists all nodes in the cluster with their status, role, age, and Kubernetes version. In a real production cluster this would show multiple nodes across different servers. Here I have one node because kind creates single-node clusters by default.

### `kubectl get namespaces`

```bash
kubectl get namespaces
```
**What this command does:** Shows all namespaces in the cluster. Namespaces are logical partitions — like separate rooms in a house — that let me isolate workloads from each other.

| Namespace | What it is |
|---|---|
| `default` | Where my own apps live if I don't specify a namespace |
| `kube-system` | Where Kubernetes' own internal components run |
| `kube-public` | Publicly readable — cluster bootstrap info lives here |
| `kube-node-lease` | Heartbeat records — how the cluster knows each node is still alive |
| `local-path-storage` | Created by kind — handles automatic local storage volumes |

### `kubectl get pods -A`

```bash
kubectl get pods -A
```
**What this command does:** `-A` stands for `--all-namespaces`. This shows every single pod running across the entire cluster. This is where I first saw all the architecture components I had studied actually running as pods in real life.

### `kubectl get pods -n kube-system`

```bash
kubectl get pods -n kube-system
```
**What this command does:** `-n` specifies a namespace. This filters to show only the system pods. Here the architecture diagram came alive — I could match every running pod to a component I had studied:

| Pod | Architecture Component | What it does |
|---|---|---|
| `etcd-devops-cluster-control-plane` | **etcd** | Stores all cluster state as key-value data |
| `kube-apiserver-devops-cluster-control-plane` | **API Server** | Receives all commands, validates, and routes them |
| `kube-scheduler-devops-cluster-control-plane` | **Scheduler** | Decides which node each new Pod runs on |
| `kube-controller-manager-devops-cluster-control-plane` | **Controller Manager** | Reconciliation loops — maintains desired state |
| `coredns-7d764666f9-b2fm8` | **CoreDNS** | DNS resolution inside the cluster |
| `coredns-7d764666f9-k28dk` | **CoreDNS replica** | Second replica for DNS reliability |
| `kube-proxy-2w8pn` | **kube-proxy** | Networking rules on the node |
| `kindnet-z6vkl` | **kindnet CNI** | Pod-to-pod networking (kind's network plugin) |

Every pod shows `READY 1/1`, `STATUS Running`, `RESTARTS 0` — my cluster is perfectly healthy with zero issues.

---

## Task 6: Practice Cluster Lifecycle

This task was about building muscle memory — delete the cluster, bring it back, and verify everything works. In real DevOps work, I need to be comfortable doing this confidently.

### 📸 Screenshot — deleting and recreating the cluster

![cluster lifecycle](screenshots/cluster-lifecycle.png)

### Step 1 — Delete the cluster

```bash
kind delete cluster --name devops-cluster
```
**What this command does:** Completely destroys the cluster — stops the Docker containers, removes the nodes, and cleans up the kubeconfig entry automatically.

**Output I got:**
```
Deleting cluster "devops-cluster" ...
Deleted nodes: ["devops-cluster-control-plane"]
```

### Step 2 — Verify it is gone

```bash
kind get clusters
```
**What this command does:** Lists all kind clusters that currently exist. Output was `No kind clusters found.` — confirms the cluster is completely gone.

```bash
kubectl get nodes
```
**What happened:** This returned the `connection refused on localhost:8080` error. And this time I understood exactly why — kubectl has no valid kubeconfig because the cluster no longer exists. This is not a bug. This is expected behaviour. Every time I see `connection refused on localhost:8080`, it means kubectl cannot find a valid kubeconfig. Either the cluster does not exist, or kubectl is running as the wrong user.

### Step 3 — Recreate the cluster

```bash
kind create cluster --name devops-cluster
```
**What this command does:** Brings the cluster back from scratch. The same steps ran again — image pull, node prep, control plane start, CNI install, StorageClass install. The whole thing came back up in under a minute.

### 📸 Screenshot — cluster back and verified

![cluster recreated](screenshots/cluster-recreated.png)

### Step 4 — Verify context and kubeconfig

```bash
kubectl get nodes
```
**What this command does:** Confirms the cluster is healthy again.
```
NAME                          STATUS   ROLES          AGE   VERSION
devops-cluster-control-plane  Ready    control-plane  78s   v1.35.1
```

```bash
kubectl config current-context
```
**What this command does:** Tells me which cluster kubectl is currently pointing to. Output: `kind-devops-cluster`. This is the most important command to run before executing anything serious — it confirms exactly where my commands are going.

```bash
kubectl config get-contexts
```
**What this command does:** Lists all clusters kubectl knows about and shows which one is active (marked with `*`).
```
CURRENT   NAME                 CLUSTER              AUTHINFO             NAMESPACE
*         kind-devops-cluster  kind-devops-cluster  kind-devops-cluster
```
In a real job, this list might show dev, staging, and prod clusters. The `*` tells me which one is active right now.

> **Mistake I made:** I accidentally typed `kubectl confing get-contexts` (typo) and got `error: unknown command "confing"`. kubectl even suggested the correct spelling. Small mistake, good reminder — always read the command before hitting Enter.

---

## What is a Kubeconfig?

A kubeconfig is a YAML file at `~/.kube/config`. It is how kubectl knows which cluster to talk to, how to authenticate, and what the default namespace is.

When I ran `kind create cluster`, kind wrote this file automatically. When I deleted the cluster, the entry got removed automatically. Before today, this file did not even exist on my machine because I had never created a cluster.

**Three sections inside kubeconfig:**

```yaml
clusters:        # list of cluster API server URLs and certificates
users:           # my credentials — certificates and tokens
contexts:        # combinations of cluster + user + namespace
current-context: kind-devops-cluster   # the currently active one
```

**Commands I now know for managing kubeconfig:**

```bash
# Where the file lives
cat ~/.kube/config

# Which cluster am I currently on?
kubectl config current-context

# All clusters I know about
kubectl config get-contexts

# Full kubeconfig file contents
kubectl config view

# Switch to a different cluster
kubectl config use-context <context-name>
```

---

## Mistakes I Made (So I Remember Them)

**1. Running kubectl as root after creating the cluster as kishor**
I created the cluster as `kishor`, so the kubeconfig was written to `/home/kishor/.kube/config`. When I switched to root and ran kubectl, it looked for `/root/.kube/config` — which did not exist — and gave the connection refused error. The fix was simple: always run kubectl as the same user who created the cluster.

**2. Trying to create `/root/.kube` without proper permissions**
I ran `mkdir /root/.kube` as a normal user and got permission denied. I did not need to do this at all — the kubeconfig was already in the right place under my home directory.

**3. Typo — `kubectl confing` instead of `kubectl config`**
kubectl caught it and suggested the correct spelling. Always read error messages carefully — they usually tell exactly what went wrong.

---

## My Key Takeaways from Day 50

- Kubernetes solves what Docker cannot — orchestrating containers across many machines at scale
- Everything goes through the **API Server** — it is the single entry point for all operations
- **etcd** is the actual database — lose etcd, lose the cluster state
- The `kube-system` namespace shows the architecture running as actual pods inside the cluster itself — I could literally see etcd, the scheduler, and the API server all running live
- `connection refused on localhost:8080` always means kubectl cannot find a kubeconfig — it is not a cluster problem
- Always run kubectl as the user who created the cluster
- Always check `kubectl config current-context` before running any important command


---

*Day 50 of #90DaysOfDevOps completed ✅*
*#DevOpsKaJosh #TrainWithShubham #Kubernetes #K8s #CloudNative*
