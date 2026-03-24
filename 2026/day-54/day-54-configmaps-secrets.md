# Day 54 – Kubernetes ConfigMaps and Secrets

> **#90DaysOfDevOps | DevOps Ka Josh | TrainWithShubham**

---

## What I Learned Today

Until today, any configuration my app needed — database URLs, ports, feature flags, passwords — would have to be hardcoded inside the container image. That means every time a value changes, I would have to rebuild the image and redeploy. That is not how production works.

Today I learned two Kubernetes resources that solve this completely:

```
ConfigMap → non-sensitive configuration  (ports, URLs, feature flags)
Secret    → sensitive data               (passwords, API keys, tokens)
```

Both allow me to keep configuration outside the container image and inject it at runtime. This follows the 12-Factor App principle — config should be separate from code.

---

## Task 1: ConfigMap from Literals

A ConfigMap is a key-value store for non-sensitive configuration. The data is stored as plain text — no encoding, no encryption.

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080
```

**`--from-literal=KEY=VALUE`** — pass key-value pairs directly in the command. No file needed.

```bash
# Human readable format
kubectl describe configmap app-config

# Full YAML structure
kubectl get configmap app-config -o yaml
```

### Screenshot

<img width="1209" height="521" alt="1" src="https://github.com/user-attachments/assets/8197781a-5c1b-4c5b-94de-d33a712d2b3e" />

<img width="1209" height="244" alt="2" src="https://github.com/user-attachments/assets/c472373f-9f71-418f-9938-8122dc15ce92" />



### What I Saw

`kubectl describe configmap app-config` showed all three keys in plain text under the `Data` section:
```
APP_DEBUG: false
APP_ENV:   production
APP_PORT:  8080
```

`kubectl get configmap app-config -o yaml` showed the same data in YAML format — completely readable, no encoding. This is exactly why ConfigMaps are only for non-sensitive data.

---

## Task 2: ConfigMap from a File

Sometimes config is not just a key-value pair — it is a full file. I can store an entire file inside a ConfigMap.

I wrote a custom Nginx config file that adds a `/health` endpoint:

```bash
vim default.conf
```

```nginx
server {
    listen 80;
    location /health {
        return 200 'healthy';
        add_header Content-Type text/plain;
    }
}
```

This config tells Nginx to listen on port 80 and return the string `healthy` when any request hits `/health`. Health check endpoints like this are standard in production — load balancers ping them to know if the app is alive.

```bash
kubectl create configmap nginx-config \
  --from-file=default.conf=default.conf
```

**`--from-file=key=filename`:**
- Left side of `=` → the key name inside the ConfigMap (this becomes the filename when mounted)
- Right side of `=` → the actual file on disk

```bash
kubectl get configmap nginx-config -o yaml
```

### Screenshot

<img width="1209" height="378" alt="3" src="https://github.com/user-attachments/assets/c890c9e9-92e4-45e6-ba6c-2aec59ac1806" />


### What I Saw

The entire file content was stored under `data.default.conf:` in the ConfigMap YAML. The pipe character `|` in YAML means multi-line string — the whole file is stored as one value.

---

## Task 3: Using ConfigMaps in a Pod

There are two ways to consume a ConfigMap inside a Pod:

```
Method 1 — Environment Variables  → for simple key-value settings
Method 2 — Volume Mount           → for full config files
```

### Pod 1 — Environment Variables (using app-config)

```bash
vim env-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo APP_ENV=$APP_ENV && echo APP_PORT=$APP_PORT && echo APP_DEBUG=$APP_DEBUG && sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
```

**`envFrom` with `configMapRef`** — injects ALL keys from the ConfigMap as environment variables automatically. I do not need to list each key individually.

```bash
kubectl apply -f env-pod.yaml
kubectl logs env-pod
```

### Screenshot

<img width="1209" height="156" alt="4" src="https://github.com/user-attachments/assets/eb587df9-c3d8-47bd-a85e-906d6e66924a" />


**Logs showed:**
```
APP_ENV=production
APP_PORT=8080
APP_DEBUG=false
```

All three values from `app-config` were injected as environment variables — exactly as stored in the ConfigMap.

---

### Pod 2 — Volume Mount (using nginx-config)

```bash
vim nginx-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: nginx-config-vol
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: nginx-config-vol
    configMap:
      name: nginx-config
```

**How volume mounting works:**
```
ConfigMap key name  →  becomes the filename inside mountPath
ConfigMap key value →  becomes the file content

So:
key:   default.conf
value: server { listen 80; location /health... }

Becomes:
/etc/nginx/conf.d/default.conf  ← file created here
content: server { listen 80; location /health... }
```

Nginx automatically reads all `.conf` files in `/etc/nginx/conf.d/` — so my custom config is picked up without any extra steps.

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods

# Test the /health endpoint
kubectl exec nginx-pod -- curl -s http://localhost/health
```

### Screenshot

<img width="1209" height="280" alt="5" src="https://github.com/user-attachments/assets/1cbd8332-4345-4236-a567-a2b772b8a20c" />


**Output:** `healthy`

The `/health` endpoint responded correctly. My ConfigMap-mounted Nginx config was working perfectly. I also noticed the first `apply` failed with a YAML parsing error — I fixed it in vim and applied again successfully.

---

## Task 4: Creating a Secret

A Secret stores sensitive data. The values are base64-encoded — which looks scrambled but is NOT encryption.

```bash
kubectl create secret generic db-credentials \
  --from-literal=DB_USER=admi \
  --from-literal=DB_PASSWORD=s3cureP@ssw0rd
```

```bash
# Inspect — values appear base64-encoded
kubectl get secret db-credentials -o yaml
```

### Screenshot

<img width="1209" height="381" alt="6" src="https://github.com/user-attachments/assets/3695075c-4596-4a9a-aeb1-9e93bb339b43" />


### What I Saw

```yaml
data:
  DB_PASSWORD: czNjdXJlUEBzc3cwcmQ=
  DB_USER: YWRtaQ==
```

The values look scrambled — but this is just base64 encoding. I proved it is not encryption by decoding them:

```bash
# Decode DB_USER
echo 'YWRtaQ==' | base64 --decode
# Output: admi

# Decode DB_PASSWORD
echo 'czNjdXJlUEBzc3cwcmQ=' | base64 --decode
# Output: s3cureP@ssw0rd

# Extract and decode in one command
kubectl get secret db-credentials -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode
# Output: s3cureP@ssw0rd
```

**Why base64 is NOT encryption:**
Anyone with access to the cluster can run `base64 --decode` and read the value. The real advantages of Secrets over ConfigMaps are:
- RBAC — I can give different teams access to ConfigMaps but not Secrets
- Secrets are stored in tmpfs (memory) on nodes — not written to disk
- Encryption at rest can be enabled at the cluster level

---

## Task 5: Using Secrets in a Pod

Just like ConfigMaps, Secrets can be consumed as environment variables or volume mounts.

```bash
vim secret-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo DB_USER=$DB_USER && cat /etc/db-credentials/DB_PASSWORD && sleep 3600"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: DB_USER
    volumeMounts:
    - name: db-creds-vol
      mountPath: /etc/db-credentials
      readOnly: true
  volumes:
  - name: db-creds-vol
    secret:
      secretName: db-credentials
```

**Two methods used at the same time:**

```
env → valueFrom → secretKeyRef  →  injects only DB_USER as environment variable
volumeMount at /etc/db-credentials →  both keys become files:
  /etc/db-credentials/DB_USER     → content: admi        (plaintext)
  /etc/db-credentials/DB_PASSWORD → content: s3cureP@ssw0rd  (plaintext)
```

**`readOnly: true`** — the container cannot write to this mount. Good security practice for credentials.

```bash
kubectl apply -f secret-pod.yaml
kubectl logs secret-pod
```

### Screenshot

<img width="1209" height="539" alt="7" src="https://github.com/user-attachments/assets/b636ff75-f6e2-4707-81c4-6c49df3241cd" />


**Logs showed:**
```
DB_USER=admi
s3cureP@ssw0rd
```

**Important observation:** The volume-mounted file content is **plaintext** — Kubernetes automatically decodes the base64 before mounting. The base64 encoding only exists in etcd storage — not in the Pod.

---

## Task 6: ConfigMap Live Update — Volume vs Environment Variables

This is the most important difference between the two consumption methods.

```
Environment Variables  →  set at Pod startup → NEVER auto-update → Pod restart needed
Volume Mounts          →  Kubernetes auto-updates within 30-60 seconds → NO restart needed
```

### Step 1 — Create ConfigMap and Pod

```bash
kubectl create configmap live-config --from-literal=message=hello
```

```bash
vim live-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: live-pod
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "while true; do cat /etc/config/message; echo ''; sleep 5; done"]
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: live-config
```

The Pod reads `/etc/config/message` every 5 seconds in a loop and prints it.

```bash
kubectl apply -f live-pod.yaml
kubectl logs live-pod -f
# Showing: hello, hello, hello...
```

### Step 2 — Update the ConfigMap

```bash
kubectl patch configmap live-config \
  --type merge \
  -p '{"data":{"message":"world"}}'
```

**`kubectl patch`** — updates only the specified field without rewriting the entire YAML.

### Screenshot

<img width="1209" height="616" alt="8" src="https://github.com/user-attachments/assets/2e6634b7-ad16-487b-9df5-855d98e6e2d2" />


### What I Saw

After the patch, logs continued showing `hello` for about 30-60 seconds, then automatically switched to `world` — without any pod restart. This is Kubernetes propagating the ConfigMap update to the volume mount automatically.

I also made a typo — typed `kubectl path` instead of `kubectl patch`. kubectl caught it and suggested the correct command. Fixed it and it worked.

---

## Task 7: Clean Up

```bash
kubectl delete pod env-pod secret-pod live-pod
kubectl delete configmap app-config nginx-config live-config
kubectl delete secret db-credentials
kubectl delete pod nginx-pod
kubectl get pod
kubectl get configmaps
kubectl get secrets
```

### Screenshot

<img width="1209" height="616" alt="9" src="https://github.com/user-attachments/assets/b2191cc7-d987-4e66-bc94-5230fb9295e8" />


---

## ConfigMap vs Secret — Side by Side

| | ConfigMap | Secret |
|---|---|---|
| Purpose | Non-sensitive config | Sensitive data |
| Storage | Plain text | base64 encoded |
| Use case | DB URL, ports, flags | Passwords, tokens, keys |
| Readable with get -o yaml | Yes, immediately | Yes, but base64 — decode needed |
| RBAC separation | Standard | Can be restricted separately |

---

## Environment Variables vs Volume Mounts

| | Environment Variables | Volume Mounts |
|---|---|---|
| How to inject | `envFrom` or `env.valueFrom` | `volumeMounts` + `volumes` |
| Best for | Simple key-value settings | Full config files |
| Auto-updates | Never — set at pod startup only | Yes — within 30-60 seconds |
| Requires pod restart to update | Yes | No |

---

## Why base64 is NOT Encryption

base64 is an encoding scheme — it converts binary data to ASCII text. It is fully reversible by anyone. Running `echo '<value>' | base64 --decode` gives back the original plaintext immediately.

Real security for Secrets comes from:
- **RBAC** — restricting who can `get` or `list` secrets
- **Encryption at rest** — encrypting the etcd database where secrets are stored
- **External secret managers** — HashiCorp Vault, AWS Secrets Manager, etc.

---

## How ConfigMap Updates Propagate

When I update a ConfigMap:
- **Volume mounts** — Kubernetes pushes the update to all Pods using that ConfigMap within 30-60 seconds. No restart needed.
- **Environment variables** — the Pod has already read the values at startup. The update does NOT reach the running Pod. A Pod restart is required.

This means for frequently changing config, volume mounts are the better choice.

---

## My Key Takeaways from Day 54

- ConfigMaps store non-sensitive config as plain text — database URLs, ports, feature flags
- Secrets store sensitive data as base64 — passwords, tokens, API keys
- base64 is encoding not encryption — anyone can decode it with one command
- `envFrom` injects all ConfigMap keys as environment variables at once
- Volume mounts turn ConfigMap keys into files — key name becomes filename, value becomes content
- Volume-mounted ConfigMaps auto-update within 30-60 seconds without pod restart
- Environment variable values are frozen at pod startup — never auto-update
- `readOnly: true` on Secret volume mounts is a security best practice
- `kubectl patch` updates a specific field without rewriting the entire YAML

---

*Day 54 of #90DaysOfDevOps completed ✅*
*#DevOpsKaJosh #TrainWithShubham #Kubernetes #K8s #ConfigMaps #Secrets #CloudNative*
