# Day 51 – Kubernetes Manifests and My First Pods

> **#90DaysOfDevOps | DevOps Ka Josh | TrainWithShubham**

---

## My Journey on Day 51

Yesterday I set up a Kubernetes cluster. Today I actually deployed something onto it. This is where Kubernetes stopped being theory and started being real. I wrote Pod manifests from scratch, got a shell inside a running container, hit my first error, debugged it myself, and cleaned everything up at the end. Here is everything I did and everything I learned.

---

## The Anatomy of a Kubernetes Manifest

Every single resource in Kubernetes — whether it is a Pod, a Deployment, a Service, or anything else — is defined using a YAML file called a **manifest**. There are four required top-level fields. If any one of them is missing, Kubernetes rejects the file completely.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

### The Four Required Fields

**`apiVersion`**
Tells Kubernetes which API group and version to use for this resource. Different resource types live in different API groups.
```yaml
apiVersion: v1        # for Pod, Service, ConfigMap
apiVersion: apps/v1   # for Deployment, ReplicaSet (Day 52)
```
If the wrong apiVersion is used, Kubernetes throws: `no matches for kind "Pod" in version "apps/v1"`

**`kind`**
The type of resource being created. Today it is `Pod`. Later it will be `Deployment`, `Service`, `ConfigMap`, `Ingress` and more.

**`metadata`**
The identity of the resource. `name` is mandatory — it must be unique within a namespace. `labels` are optional key-value pairs used for organizing and filtering resources.
```yaml
metadata:
  name: nginx-pod           # unique name inside the namespace
  labels:
    app: nginx              # key=value — used for filtering
    environment: production
```

**`spec`**
The actual desired state — what should run, how it should run. For a Pod, this means which containers, which images, which ports, which commands, and which environment variables.

---

## What is a Pod?

A Pod is the **smallest deployable unit** in Kubernetes. Containers do not run directly on a node — they run inside a Pod. Think of a Pod as a wrapper around one or more containers that share the same network and storage.

```
Kubernetes Cluster
└── Node
    └── Pod                ← smallest deployable unit
        └── Container(s)   ← actual running process
```

In most real-world cases, one Pod contains exactly one container. Multiple containers in one Pod is an advanced pattern called a sidecar — that comes later.

---

## Task 1: My First Pod — Nginx

### The Manifest I Wrote

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Commands I Ran

```bash
# Create the directory for Day 51 work
mkdir Day-51
cd Day-51

# Write the manifest
vim nginx-pod.yml

# Apply the manifest to the cluster
kubectl apply -f nginx-pod.yml

# Check the pod status
kubectl get pod

# Check with extra details — IP address and node
kubectl get pods -o wide
```

### 📸 Screenshot — Nginx Pod Created and Running


<img width="1051" height="292" alt="1 Pod — Nginx" src="https://github.com/user-attachments/assets/313b9922-04f8-4f9b-8804-09b77745b3e7" />


**What I observed:**

The first time I ran `kubectl apply -f nginx-pod.yml` I got this error:
```
error validating "nginx-pod.yml": error validating data:
apiVersion not set
```
This happened because I forgot to include `apiVersion: v1` in my YAML. I opened the file again with `vim`, added the missing field, and applied it again. This time it worked:
```
pod/nginx-pod created
```

`kubectl get pod` showed:
```
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          38s
```

`kubectl get pods -o wide` showed the Pod IP and which node it landed on:
```
NAME        READY   STATUS    IP           NODE
nginx-pod   1/1     Running   10.244.0.5   devops-cluster-control-plane
```

The `-o wide` flag is useful when debugging networking issues — it shows exactly where the Pod is running and what IP it has.

### Describing the Pod

```bash
kubectl describe pod nginx-pod
```

### 📸 Screenshot — kubectl describe pod nginx-pod


<img width="1051" height="619" alt="2 describe" src="https://github.com/user-attachments/assets/039113fa-ffe6-453a-8180-a18ac66d499b" />


`describe` gives a full breakdown of the Pod — its current state, conditions, volumes, and most importantly the **Events** section at the bottom. The events show exactly what happened step by step:

```
Scheduled   → Successfully assigned to devops-cluster-control-plane
Pulling     → Pulling image "nginx:latest"
Pulled      → Successfully pulled image in 12.055s
Created     → Container created
Started     → Container started
```

This Events section is the most useful thing for debugging. When a Pod is stuck or crashing, `kubectl describe` is always the first command to run.

### Reading the Logs

```bash
kubectl logs nginx-pod
```

### 📸 Screenshot — kubectl logs nginx-pod


<img width="1051" height="365" alt="3 logs" src="https://github.com/user-attachments/assets/47fe5ad4-2df4-4d43-96fc-87d2d23a4f93" />


The logs show Nginx startup output — configuration loading, IPv6 setup, worker processes starting. These are the container's stdout and stderr. In production, logs are how I debug what is happening inside a running container without ever touching the server directly.

### Getting a Shell Inside the Container

```bash
kubectl exec -it nginx-pod -- /bin/bash
```

`-it` means interactive terminal — it opens a live shell inside the running container. Once inside:

```bash
curl localhost:80
exit
```

### 📸 Screenshot — curl localhost:80 from inside the container


<img width="1051" height="526" alt="4  curl localhost:80
" src="https://github.com/user-attachments/assets/76c14f1e-8ff3-4e80-8b2e-af3bd5660af2" />


The HTML output confirmed Nginx is running and responding on port 80:
```html
Welcome to nginx!
If you see this page, nginx is successfully installed and working.
```

This is the Nginx welcome page being served from inside the container. The Pod is fully working.

---

## Task 2: BusyBox Pod

BusyBox is a tiny ~1MB Linux image. Unlike Nginx, it has no long-running server process. If a container process exits, the Pod goes into `CrashLoopBackOff`. This is why the `command` field is needed here — to keep the container alive.

### The Manifest I Wrote

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```

The `command` field runs a shell that first prints a message then sleeps for 3600 seconds (1 hour). Without `sleep 3600`, the container would exit in milliseconds and Kubernetes would keep trying to restart it — that is `CrashLoopBackOff`.

### Commands I Ran

```bash
vim busybox-pod.yaml
kubectl apply -f busybox-pod.yaml
kubectl get pods
kubectl logs busybox-pod
```

### 📸 Screenshot — BusyBox Pod Running


<img width="1051" height="175" alt="BusyBox" src="https://github.com/user-attachments/assets/619bb7dd-879e-4102-af4e-fa4fefd3d82f" />


`kubectl get pods` confirmed both pods running:
```
NAME          READY   STATUS    RESTARTS   AGE
busybox-pod   1/1     Running   0          10s
nginx-pod     1/1     Running   0          11m
```

`kubectl logs busybox-pod` showed:
```
Hello from BusyBox
```

The `echo` command ran at startup and the message was captured in the logs. The container is still alive because `sleep 3600` is keeping it running.

---

## Task 3: Imperative vs Declarative

This is one of the most important concepts in Kubernetes — and in DevOps in general.

### Declarative (what I have been doing)

```bash
kubectl apply -f nginx-pod.yml
```

I write a YAML file describing the **desired state**. Kubernetes figures out how to achieve it. If I run the same command again nothing changes — it is idempotent. All changes are tracked in files that can be stored in Git. This is what production environments use.

### Imperative (direct commands)

```bash
kubectl run redis-pod --image=redis:latest
```

No YAML file. Just a direct command. Fast for quick testing but not suitable for production because nothing is tracked or repeatable.

### The Dry Run Trick

```bash
kubectl run test-pod --image=nginx --dry-run=client -o yaml
```

`--dry-run=client` generates the YAML without actually creating anything. The output can be saved to a file and customized:

```bash
kubectl run test-pod --image=nginx --dry-run=client -o yaml > test-pod.yaml
```

### 📸 Screenshot — Imperative vs Declarative


<img width="1051" height="334" alt="Imperative vs Declarative" src="https://github.com/user-attachments/assets/26e9d9cf-3b97-4fd0-b749-8904f5573a95" />


The dry-run output showed the auto-generated YAML:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test-pod
  name: test-pod
spec:
  containers:
  - image: nginx
    name: test-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Comparing this with my hand-written `nginx-pod.yml` — the structure is the same but Kubernetes added extra fields like `dnsPolicy`, `restartPolicy`, and `resources: {}` automatically. These are default values that Kubernetes fills in even when I do not specify them.

---

## Task 4: Validate Before Applying

Before applying any manifest, it is good practice to validate it first — especially before touching a production cluster.

```bash
# Client-side validation — checks YAML syntax locally
kubectl apply -f nginx-pod.yml --dry-run=client

# Server-side validation — checks against the actual API server
kubectl apply -f nginx-pod.yml --dry-run=server
```

**Difference between the two:**
- `--dry-run=client` validates locally without contacting the cluster — catches YAML syntax errors
- `--dry-run=server` sends the request to the API server but does not persist it — catches semantic errors that only the cluster knows about

### 📸 Screenshot — Validation


<img width="1051" height="150" alt="Validate Before Applying" src="https://github.com/user-attachments/assets/67fc4b8d-1ebc-4c87-a6d7-5298e3b41b83" />


**What I observed:**

`--dry-run=client` output:
```
pod/nginx-pod configured (dry run)
```

`--dry-run=server` output:
```
pod/nginx-pod unchanged (server dry run)
```

When I intentionally deleted the `nginx-pod.yml` file and tried to validate it:
```
error: the path "nginx-pod.yml" does not exist
```

Kubernetes immediately caught that the file was missing. This validation step would have prevented me from accidentally applying a broken or missing manifest to a live cluster.

---

## Task 5: Pod Labels and Filtering

Labels are key-value pairs attached to Kubernetes resources. Kubernetes itself does not care what labels say — they are purely for humans and for **selectors** (which Deployments and Services use to find their Pods).

### Commands I Ran

```bash
# See all pods with their labels
kubectl get pods --show-labels

# Filter pods by label
kubectl get pods -l app=nginx
kubectl get pods -l environment=dev

# Add a new label to an existing pod
kubectl label pod nginx-pod environment=production

# Verify the label was added
kubectl get pods --show-labels

# Remove a label (add a minus sign after the key)
kubectl label pod nginx-pod environment-
```

### 📸 Screenshot — Labels and Filtering


<img width="1051" height="372" alt="Labels aur Filtering" src="https://github.com/user-attachments/assets/49837662-70b0-459a-a1f1-8214ceddfb2c" />


**What I observed:**

`kubectl get pods --show-labels` showed all labels on each pod:
```
NAME          LABELS
busybox-pod   app=busybox,environment=dev
nginx-pod     app=nginx
redis-pod     run=redis-pod
```

After adding `environment=production` to nginx-pod:
```
NAME          LABELS
nginx-pod     app=nginx,environment=production
```

After removing it with `kubectl label pod nginx-pod environment-`:
```
pod/nginx-pod unlabeled
```

The `-` suffix on a label key is how labels are removed in Kubernetes.

### Third Pod — myapp-pod with 3 Labels

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    environment: staging
    team: backend
spec:
  containers:
  - name: myapp
    image: nginx:latest
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f myapp-pod.yaml
kubectl get pods -l team=backend
kubectl get pods -l environment=staging
kubectl get pods -l app=myapp,team=backend
```

### 📸 Screenshot — myapp-pod with Multiple Label Filters


<img width="1051" height="303" alt="myapp-pod" src="https://github.com/user-attachments/assets/e90e93bd-6abc-4133-a025-8e673a1f8439" />


**What I observed:**

```bash
kubectl get pods -l team=backend
# NAME        READY   STATUS    AGE
# myapp-pod   1/1     Running   64s

kubectl get pods -l environment=staging
# NAME        READY   STATUS    AGE
# myapp-pod   1/1     Running   80s

kubectl get pods -l app=myapp,team=backend
# NAME        READY   STATUS    AGE
# myapp-pod   1/1     Running   86s
```

Multiple labels in one filter act as AND — the pod must match all the specified labels to appear in the results.

---

## Task 6: Clean Up

```bash
kubectl delete pod nginx-pod
kubectl delete pod busybox-pod
kubectl delete pod myapp-pod

# Verify everything is gone
kubectl get pods
```

### 📸 Screenshot — Clean Up


<img width="1051" height="128" alt="Clean Up" src="https://github.com/user-attachments/assets/a2a62544-874a-43fb-a268-e9cb08e20bf9" />


**Output:**
```
pod "nginx-pod" deleted from default namespace
pod "busybox-pod" deleted from default namespace
pod "myapp-pod" deleted from default namespace
```

### The Most Important Lesson from Clean Up

When a standalone Pod is deleted, **it is gone forever**. There is no controller watching it and no mechanism to bring it back. This is exactly why production workloads are never run as bare Pods — they use **Deployments** (Day 52). A Deployment has a controller that constantly watches and automatically recreates any Pod that dies or gets deleted.

---

## Mistake I Made

When I first applied `nginx-pod.yml`, I forgot to include `apiVersion: v1` in the file. Kubernetes immediately rejected it:
```
error validating "nginx-pod.yml": error validating data:
apiVersion not set
```
The fix was simple — open the file, add `apiVersion: v1` at the top, save, and apply again. The lesson: always include all four required fields. If a manifest is rejected, read the error message carefully — Kubernetes always tells exactly what is wrong.

---

## Summary — What I Learned on Day 51

| Concept | What I Learned |
|---|---|
| **Manifest** | Every K8s resource is defined in YAML with 4 required fields: apiVersion, kind, metadata, spec |
| **Pod** | Smallest deployable unit — a wrapper around one or more containers |
| **kubectl apply** | Declarative command — applies desired state from a YAML file |
| **kubectl run** | Imperative command — creates a resource directly without a file |
| **kubectl describe** | Shows full details and Events — first tool for debugging |
| **kubectl logs** | Shows container stdout/stderr output |
| **kubectl exec -it** | Opens a live shell inside a running container |
| **--dry-run=client** | Validates manifest without creating anything |
| **Labels** | Key-value pairs for organizing and filtering resources |
| **Standalone Pod** | Deleted forever when removed — use Deployments in production |

---

> **Coming up on Day 52:** I will learn about Deployments — the resource that manages Pods automatically, handles rolling updates, and brings self-healing to my workloads.

---

*Day 51 of #90DaysOfDevOps completed ✅*
*#DevOpsKaJosh #TrainWithShubham #Kubernetes #K8s #Pods #CloudNative*
