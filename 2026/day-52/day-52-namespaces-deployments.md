# Day 52 – Kubernetes Namespaces and Deployments

> **#90DaysOfDevOps | DevOps Ka Josh | TrainWithShubham**

---

## What I Learned Today

On Day 51, I created standalone Pods. The big problem was — delete a Pod and it is gone forever. Nobody brings it back. Today I fixed that problem with Deployments. I also learned Namespaces, which let me organise and isolate resources inside the same cluster.

---

## What are Namespaces and Why I Need Them

A Namespace is a logical partition inside a Kubernetes cluster. Think of it like separate folders inside one big cluster. Resources in one namespace do not interfere with resources in another namespace.

The same cluster is shared between multiple teams — dev, staging, production. Without namespaces, everyone's pods and deployments would mix together and cause chaos. Namespaces give each team their own isolated space.

---

## Task 1: Exploring Default Namespaces

```bash
kubectl get namespaces
kubectl get pods -n kube-system
```

### Screenshot

<img width="1138" height="360" alt="1" src="https://github.com/user-attachments/assets/7e09525e-cfa7-4041-99e0-7ab8b13535a3" />


### What I Saw

Kubernetes comes with 4 built-in namespaces by default:

| Namespace | What lives here |
|---|---|
| `default` | Where my resources go if I do not specify a namespace |
| `kube-system` | Kubernetes internal components — API server, etcd, scheduler |
| `kube-public` | Publicly readable cluster information |
| `kube-node-lease` | Node heartbeat tracking — each node says "I am alive" here |

Inside `kube-system` I found **8 pods** all running — these are the same architecture components I studied on Day 50, still keeping my cluster alive.

---

## Task 2: Creating Custom Namespaces

```bash
# Imperative method — direct command
kubectl create namespace dev
kubectl create namespace staging

kubectl get namespaces
```

I also created a namespace from a YAML manifest — the declarative method:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

```bash
kubectl apply -f namespace.yaml
kubectl get namespaces
```

Then I ran pods inside specific namespaces:

```bash
kubectl run nginx-dev --image=nginx:latest -n dev
kubectl run nginx-staging --image=nginx:latest -n staging
```

### Screenshot

<img width="1138" height="620" alt="2" src="https://github.com/user-attachments/assets/aae29285-98ee-465b-8599-c4ca32bc8a85" />

<img width="1138" height="483" alt="3" src="https://github.com/user-attachments/assets/5ca81232-223b-4840-9443-46e44798c2fb" />


### I Learned

```bash
kubectl get pods        # shows NOTHING — only looks at default namespace
kubectl get pods -A     # shows EVERYTHING across all namespaces
```

This is the most important thing about namespaces — `kubectl get pods` by default only shows the `default` namespace. I must use `-n <namespace>` to look inside a specific namespace, or `-A` to see everything everywhere.

---

## Task 3: Creating My First Deployment

A Deployment tells Kubernetes — "I want exactly 3 Pods running at all times." If a Pod crashes or gets deleted, the Deployment controller automatically creates a new one to replace it. This is the real way to run applications in production.

The internal structure looks like this:

```
Deployment              ← I manage this
└── ReplicaSet          ← Deployment creates this automatically
    ├── Pod 1           ← ReplicaSet creates and watches these
    ├── Pod 2
    └── Pod 3
```

### My Deployment Manifest — nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
```

### Explanation of Every Field

| Field | What it does |
|---|---|
| `apiVersion: apps/v1` | Deployment lives in the `apps` API group — not core `v1` like Pod |
| `kind: Deployment` | Resource type is Deployment |
| `namespace: dev` | This Deployment lives in the dev namespace |
| `replicas: 3` | I want exactly 3 Pods running at all times |
| `selector.matchLabels` | How the Deployment finds its Pods — must exactly match `template.metadata.labels` |
| `template` | The blueprint used to create every Pod |
| `image: nginx:1.24` | Specific version — so I can do a rolling update to 1.25 later |

```bash
mkdir Day-52
cd Day-52
vim nginx-deployment.yaml
kubectl apply -f nginx-deployment.yaml
kubectl get deployment -n dev
kubectl get pods -n dev
kubectl get replicasets -n dev
```

### Screenshot

<img width="1138" height="408" alt="4" src="https://github.com/user-attachments/assets/e0e5844f-8d43-4e19-8932-c4a837e4377a" />


### I Saw

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           32s
```

- `READY 3/3` — 3 pods exist and all 3 are healthy
- `UP-TO-DATE 3` — all 3 pods are running the latest configuration
- `AVAILABLE 3` — all 3 pods are ready to serve traffic

The pods were named `nginx-deployment-5d9c84579f-xxxxx` — Deployment name plus a random hash. Kubernetes generates these names automatically. I also saw one ReplicaSet was created automatically behind the scenes.

---

## Task 4: Self-Healing — Delete a Pod and Watch It Come Back

This is the biggest difference between a standalone Pod and a Deployment.

```bash
kubectl get pods -n dev

# I deleted this pod
kubectl delete pod nginx-deployment-5d9c84579f-2cg5z -n dev

# Immediately checked again
kubectl get pods -n dev
```

### Screenshot

<img width="1138" height="310" alt="5" src="https://github.com/user-attachments/assets/16170bf1-8352-4efb-b60e-1afb1f94508d" />


### What Happened

The pod `nginx-deployment-5d9c84579f-2cg5z` was deleted. Within seconds, a brand new pod `nginx-deployment-5d9c84579f-77kf2` appeared with `AGE: 5s`. The new pod has a completely different name — it is not the same pod restarted, it is a brand new pod.

The Deployment controller detected that only 2 of 3 desired replicas existed and immediately created a replacement. This is self-healing in action. A standalone Pod would be gone forever — a Deployment pod comes back automatically.

---

## Task 5: Scaling the Deployment

### Imperative Scaling — Direct Command

```bash
# Scale up to 5
kubectl scale deployment nginx-deployment --replicas=5 -n dev
kubectl get pods -n dev
# 5 pods appeared — 2 new ones in ContainerCreating state

# Scale down to 2
kubectl scale deployment nginx-deployment --replicas=2 -n dev
kubectl get pods -n dev
# 3 pods went into Terminating state and disappeared
```

### Declarative Scaling — Edit the YAML

```bash
vim nginx-deployment.yaml
# changed replicas: 2 to replicas: 4
kubectl apply -f nginx-deployment.yaml
kubectl get pods -n dev
# 2 new pods appeared
```

### Screenshot

<img width="1138" height="360" alt="6" src="https://github.com/user-attachments/assets/33bd31a9-d091-491a-8ff1-038563dcce91" />

<img width="1138" height="526" alt="7" src="https://github.com/user-attachments/assets/1bc4fe83-1df8-4e75-b45a-405a6c4a9baa" />


### Difference Between the Two Methods

| Method | Command | When to use |
|---|---|---|
| Imperative | `kubectl scale --replicas=5` | Quick fix, testing, emergency |
| Declarative | Edit YAML + `kubectl apply` | Always in production — YAML is the source of truth |

The problem with imperative scaling is that my YAML file still shows the old replica count. Next time I apply the file, it will overwrite my change. In production, I always edit the YAML file and apply it — so the file in Git matches what is actually running.

---

## Task 6: Rolling Update and Rollback

### Rolling Update

A rolling update replaces pods one by one — not all at once. At no point are all pods down. Traffic keeps flowing to the old pods while new ones come up. This means zero downtime.

```
Before:  [nginx:1.24] [nginx:1.24] [nginx:1.24]
Step 1:  [nginx:1.25] [nginx:1.24] [nginx:1.24]
Step 2:  [nginx:1.25] [nginx:1.25] [nginx:1.24]
After:   [nginx:1.25] [nginx:1.25] [nginx:1.25]
```

```bash
# Update the image version
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev

# Watch the rollout happen in real time
kubectl rollout status deployment/nginx-deployment -n dev

# See revision history
kubectl rollout history deployment/nginx-deployment -n dev
```

### Screenshot

<img width="1138" height="416" alt="8" src="https://github.com/user-attachments/assets/29520671-04a7-4dbf-8810-4b2845d3e040" />


### What I Saw During the Rollout

```
Waiting for deployment rollout to finish: 3 out of 4 new replicas have been updated...
Waiting for deployment rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
```

The history showed 2 revisions — Revision 1 was nginx:1.24 and Revision 2 was nginx:1.25.

### Rollback

Something went wrong with the new version? One command brings everything back to the previous version.

```bash
kubectl rollout undo deployment/nginx-deployment -n dev
kubectl rollout status deployment/nginx-deployment -n dev

# Verify which image is now running
kubectl describe deployment nginx-deployment -n dev | grep Image
```

### Screenshot

<img width="1138" height="92" alt="9" src="https://github.com/user-attachments/assets/62ca620d-1826-405d-aba8-8e18cb0170d3" />


### What I Saw After Rollback

```
Image: nginx:1.24
```

Back to version 1.24. Kubernetes kept the old ReplicaSet around specifically for this purpose — instant rollback without pulling any new images.

---

## Task 7: Clean Up

```bash
kubectl delete deployment nginx-deployment -n dev
kubectl delete pod nginx-dev -n dev
kubectl delete pod nginx-staging -n staging
kubectl delete namespace dev staging production

kubectl get namespaces
kubectl get pods -A
```

### Screenshot

<img width="1138" height="548" alt="10" src="https://github.com/user-attachments/assets/346a58c9-0e30-405f-a081-ff0c5ee92031" />


### What Happened

All three namespaces — `dev`, `staging`, `production` — were deleted. Everything inside them was deleted along with them. Only the default Kubernetes namespaces remained. The only pod left running was `redis-pod` in the `default` namespace from Day 51 practice.

**Most important rule:** Deleting a namespace deletes everything inside it — all pods, deployments, services, configmaps, everything. In production this can be catastrophic. I must always double check which namespace I am targeting before running delete commands.

---

## Key Differences — Standalone Pod vs Deployment Pod

| Situation | Standalone Pod | Deployment Pod |
|---|---|---|
| Pod deleted | Gone forever | New pod created automatically |
| Pod crashes | Gone forever | New pod created automatically |
| Want 3 copies | Must create 3 separate YAML files | Just set `replicas: 3` |
| Rolling update | Not possible | Built in — zero downtime |
| Rollback | Not possible | One command — `kubectl rollout undo` |

---

## How Scaling Works

**Imperative** — `kubectl scale deployment <name> --replicas=N`
Fast and easy. Good for quick testing. But the YAML file does not get updated — dangerous in teams.

**Declarative** — Edit `replicas` in the YAML file and run `kubectl apply -f`
The correct production approach. The YAML file always reflects what is actually running. This file lives in Git and the whole team can see it.

---

## How Rolling Updates and Rollbacks Work

A rolling update gradually replaces old pods with new ones — one by one. Kubernetes never terminates an old pod until the new one is healthy. This guarantees zero downtime during updates.

A rollback undoes the last update instantly. Kubernetes keeps the previous ReplicaSet around for exactly this reason. `kubectl rollout undo` switches traffic back to the old ReplicaSet within seconds.

---

## My Key Takeaways from Day 52

- Namespaces are logical partitions — they isolate resources from each other inside one cluster
- `kubectl get pods` only shows the `default` namespace — always use `-n` or `-A`
- A Deployment maintains desired state forever — deleted pods come back automatically
- The `selector.matchLabels` must exactly match `template.metadata.labels` — this is how the Deployment finds its pods
- Scaling imperatively is fast but does not update the YAML — always edit the YAML in production
- Rolling updates replace pods one by one — zero downtime
- Rollbacks are instant — Kubernetes keeps old ReplicaSets for this exact reason
- Deleting a namespace deletes everything inside it — be very careful in production

---


*Day 52 of #90DaysOfDevOps completed ✅*
*#DevOpsKaJosh #TrainWithShubham #Kubernetes #K8s #CloudNative*
