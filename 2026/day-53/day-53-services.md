# Day 53 – Kubernetes Services

> **#90DaysOfDevOps | DevOps Ka Josh | TrainWithShubham**

---

## What I Learned Today

On Day 52, I had Deployments running multiple Pods. But there was a big problem — every Pod gets its own IP address, and that IP changes every time the Pod restarts or gets replaced. On top of that, if I have 3 Pods running, which IP do I send traffic to?

Today I solved both problems with **Services**. A Service gives Pods a stable network endpoint that never changes, and automatically load balances traffic across all matching Pods.

---

## Why Services Exist

```
Without Service:
Client → Pod1 IP (10.244.0.6) ?
Client → Pod2 IP (10.244.0.7) ?   ← Pod IPs change on restart
Client → Pod3 IP (10.244.0.8) ?

With Service:
Client → Service (10.96.3.102) → Pod1
                               → Pod2   ← stable IP, load balanced
                               → Pod3
```

A Service solves two problems at once:
- **Stable IP** — the Service IP never changes even if Pods restart and get new IPs
- **Load balancing** — traffic is automatically distributed across all healthy Pods

---

## The Three Service Types

```
ClusterIP    → only reachable from inside the cluster
NodePort     → reachable from outside via a port on the Node
LoadBalancer → reachable from outside via a cloud load balancer
```

They build on each other:
```
LoadBalancer
└── NodePort
    └── ClusterIP
```
A LoadBalancer Service automatically has a NodePort and ClusterIP inside it.

---

## Task 1: Deploy the Application

```bash
vim app-deployment.yaml
kubectl apply -f app-deployment.yaml
kubectl get pods -o wide
```

### app-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### Screenshot

<img width="1222" height="218" alt="1" src="https://github.com/user-attachments/assets/6b8ff80c-2fda-4db6-afba-bf4aabdcbd03" />



### What I Saw

`kubectl get pods -o wide` — the `-o wide` flag adds extra columns including **IP** and **NODE**.

```
NAME                      IP           NODE
web-app-6cffb4b956-9llsx  10.244.0.8   devops-cluster-control-plane
web-app-6cffb4b956-dv7bj  10.244.0.7   devops-cluster-control-plane
web-app-6cffb4b956-xbrp4  10.244.0.6   devops-cluster-control-plane
```

Each Pod got a unique IP. These IPs are unstable — they change when a Pod restarts. A Service replaces the need to track these IPs directly.

---

## Task 2: ClusterIP Service

**ClusterIP** is the default Service type. It provides a stable internal IP that is only reachable from inside the cluster. External traffic cannot reach it.

**Use case:** Internal microservice communication — when one service inside the cluster needs to talk to another.

```bash
vim clusterip-service.yaml
kubectl apply -f clusterip-service.yaml
kubectl get services
```

### clusterip-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

### Explanation of Every Field

| Field | What it does |
|---|---|
| `type: ClusterIP` | Only accessible from inside the cluster |
| `selector: app: web-app` | Routes traffic to all Pods with the label `app: web-app` — must match Pod labels exactly |
| `port: 80` | The port the Service listens on |
| `targetPort: 80` | The port on the Pod where traffic is forwarded — these two numbers do not have to match |

### Screenshot

<img width="1222" height="131" alt="2" src="https://github.com/user-attachments/assets/e6ca5590-e6f3-4685-a3dc-04669d0892ca" />


**Output:**
```
NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP   7d21h
web-app-clusterip   ClusterIP   10.96.3.102    <none>        80/TCP    7s
```

`CLUSTER-IP: 10.96.3.102` — this is the stable IP. Even if all 3 Pods restart and get new IPs, this IP stays the same.

### Testing ClusterIP from Inside the Cluster

ClusterIP is not reachable from my laptop terminal — only from inside the cluster. So I ran a temporary test Pod:

```bash
kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh
```

**What each flag does:**
- `--rm` — automatically delete this Pod when I exit
- `-it` — interactive terminal so I can type commands inside
- `--restart=Never` — run as a one-time Pod, not a Deployment
- `-- sh` — start a shell inside the container

Inside the Pod:
```bash
wget -qO- http://web-app-clusterip
exit
```

### Screenshot

<img width="1222" height="530" alt="3" src="https://github.com/user-attachments/assets/3fbdba86-31e7-471f-8a6f-6d00105041b0" />


The full Nginx welcome page HTML came back. The Service received the request and forwarded it to one of the 3 Pods — ClusterIP working perfectly.

### Checking Endpoints

```bash
kubectl get endpoints web-app-clusterip
```

### Screenshot

<img width="1222" height="105" alt="4" src="https://github.com/user-attachments/assets/afd1dc35-56f5-45e8-b1e9-ac893e8f11b9" />


**Output:**
```
NAME                ENDPOINTS                                    AGE
web-app-clusterip   10.244.0.6:80,10.244.0.7:80,10.244.0.8:80   2m52s
```

Endpoints show exactly which Pod IPs the Service routes traffic to — the same 3 IPs from `kubectl get pods -o wide`. When a Pod restarts and gets a new IP, the Endpoints list updates automatically while the Service IP stays the same.

Note: There was a warning `v1 Endpoints is deprecated in v1.33+` — Kubernetes is moving to a newer API called EndpointSlice, but the old command still works fine.

---

## Task 3: DNS Discovery

Kubernetes has a built-in DNS server called **CoreDNS**. Every Service automatically gets a DNS entry:

```
<service-name>.<namespace>.svc.cluster.local
```

My Service gets: `web-app-clusterip.default.svc.cluster.local`

```bash
kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside the Pod:
```bash
nslookup web-app-clusterip
nslookup web-app-clusterip.default.svc.cluster.local
exit
```

### Screenshot

<img width="1222" height="617" alt="5" src="https://github.com/user-attachments/assets/4fa990d8-4ac5-4f14-938d-4168d2448350" />


### What I Saw

```
Server:   10.96.0.10          ← CoreDNS server IP
Address:  10.96.0.10:53

Name:     web-app-clusterip.default.svc.cluster.local
Address:  10.96.3.102         ← matches CLUSTER-IP exactly
```

DNS resolved `web-app-clusterip` to `10.96.3.102` — the same ClusterIP from `kubectl get services`. This means inside the cluster, I never need to remember IP addresses. I just use the Service name.

### Note on Full DNS wget Error

When I tried `wget -qO- http://web-app-clusterip.default.svc.cluster.local` I got a TLS error. This is a known bug in newer versions of busybox wget — it incorrectly attempts a TLS handshake on plain HTTP. The Service is working fine — confirmed by nslookup and the short name wget which worked perfectly.

**When to use which name:**
- Same namespace → short name: `http://web-app-clusterip`
- Different namespace → full name: `http://web-app-clusterip.default.svc.cluster.local`

---

## Task 4: NodePort Service

**NodePort** opens a specific port on every Node. External traffic can reach the Service by hitting `<NodeIP>:<NodePort>`.

**Use case:** Development, testing, or on-premise setups without a cloud load balancer.

**Traffic flow:**
```
External → Node IP:30080 → Service → Pod:80
```

```bash
vim nodeport-service.yaml
kubectl apply -f nodeport-service.yaml
kubectl get services
kubectl get nodes -o wide
```

### nodeport-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

**`nodePort: 30080`** — the port opened on the Node. Must be between 30000-32767. If not specified, Kubernetes picks a random port in that range automatically.

### Screenshot

<img width="1222" height="231" alt="7" src="https://github.com/user-attachments/assets/1a4ec747-cb60-4d01-ae58-570e81bdfca3" />

**Output:**
```
NAME               TYPE       CLUSTER-IP      PORT(S)        AGE
web-app-nodeport   NodePort   10.96.73.157    80:30080/TCP   13s
```

`80:30080/TCP` — Service listens on port 80, Node exposes port 30080.

`kubectl get nodes -o wide` showed Node internal IP: `172.18.0.2`. External access would be: `curl http://172.18.0.2:30080`

---

## Task 5: LoadBalancer Service

**LoadBalancer** is used in production on cloud platforms. The cloud provider provisions a real external load balancer with a public IP.

On kind (local cluster), `EXTERNAL-IP` shows `<pending>` — no cloud provider exists to create the real load balancer. This is completely expected, not an error.

```bash
vim loadbalancer-service.yaml
kubectl apply -f loadbalancer-service.yaml
kubectl get services
kubectl describe service web-app-loadbalancer
```

### loadbalancer-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

### Screenshot

<img width="1222" height="499" alt="8" src="https://github.com/user-attachments/assets/61431e81-3cdb-48ae-a239-43c02755d34f" />


**`kubectl describe service web-app-loadbalancer` showed:**
```
Type:        LoadBalancer
IP:          10.96.235.99        ← has a ClusterIP
NodePort:    31483/TCP           ← has a NodePort (auto-assigned)
Endpoints:   10.244.0.7:80, 10.244.0.6:80, 10.244.0.8:80
```

This confirmed that a LoadBalancer Service contains all three layers — ClusterIP, NodePort, and LoadBalancer on top.

---

## Task 6: All Services and Endpoints Side by Side

```bash
kubectl get endpoints
```

### Screenshot

<img width="1222" height="149" alt="9" src="https://github.com/user-attachments/assets/3f8278ee-5f5e-43d5-92c2-62b9ae542df8" />


All three Services showed the exact same Endpoints — `10.244.0.6:80, 10.244.0.7:80, 10.244.0.8:80`. All three Services route to the same 3 Pods, just through different access methods.

### Service Types Comparison

| Type | Accessible From | EXTERNAL-IP | Real World Use |
|---|---|---|---|
| `ClusterIP` | Inside cluster only | none | Internal microservice communication |
| `NodePort` | Outside via Node IP + Port | none | Development, testing, on-premise |
| `LoadBalancer` | Outside via public IP | pending locally / real IP on cloud | Production on AWS, GCP, Azure |

### The Most Important Rule

The `selector` in a Service must exactly match the `labels` on the Pods:

```yaml
# Service selector
selector:
  app: web-app

# Pod label — must be identical
labels:
  app: web-app
```

If they do not match, Endpoints will be empty and traffic goes nowhere. Always check `kubectl get endpoints <service-name>` when a Service is not working.

---

## Task 7: Clean Up

```bash
kubectl delete -f app-deployment.yaml
kubectl delete -f clusterip-service.yaml
kubectl delete -f nodeport-service.yaml
kubectl delete -f loadbalancer-service.yaml

kubectl get pods
kubectl get services
```

### Screenshot

<img width="1222" height="258" alt="10" src="https://github.com/user-attachments/assets/f7edecc3-ca6b-4f7c-9bb0-8b892d48dea3" />


All Services and the Deployment were deleted. Only the built-in `kubernetes` Service remained — confirming a clean cluster state.

---

## What Endpoints Are and How to Inspect Them

An Endpoint is the live list of Pod IPs that a Service is currently routing traffic to. Kubernetes maintains this list automatically — when a Pod is added, removed, or replaced, the Endpoints update instantly.

```bash
# Check which Pods a Service is routing to
kubectl get endpoints <service-name>

# If Endpoints shows <none> → selector does not match any Pod labels → fix the labels
```

---

## How Kubernetes DNS Works

CoreDNS runs inside `kube-system` namespace and gives every Service an automatic DNS entry. Inside any Pod, I can reach any Service by name — no IP needed.

```
Short name (same namespace):    http://web-app-clusterip
Full name (across namespaces):  http://web-app-clusterip.default.svc.cluster.local
```

This is called **service discovery** — applications use Service names as hostnames, and CoreDNS resolves them to the stable ClusterIP automatically.

---

## My Key Takeaways from Day 53

- Pod IPs are unstable — they change on restart. Service IPs are stable — they never change
- `selector` in a Service must exactly match `labels` on Pods — if not, Endpoints will be empty
- `kubectl get endpoints` is the first debugging tool when a Service is not working
- ClusterIP = internal only, NodePort = external via node port, LoadBalancer = external via cloud
- LoadBalancer contains NodePort which contains ClusterIP — they build on each other
- Every Service gets an automatic DNS entry — `<name>.<namespace>.svc.cluster.local`
- `EXTERNAL-IP: <pending>` on a local cluster is completely expected — not an error
- `port` is what the Service listens on, `targetPort` is what the Pod listens on — they can be different

---


*Day 53 of #90DaysOfDevOps completed ✅*
*#DevOpsKaJosh #TrainWithShubham #Kubernetes #K8s #Services #CloudNative*
