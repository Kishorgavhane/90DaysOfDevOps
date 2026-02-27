# Day 33 – Docker Compose: Multi-Container Basics

## TASK 1 – Check & Verify Compose

**Check Version**
```bash
docker compose version
```
If it shows version. then ok
but
If not installed:
```bash
sudo apt install docker-compose-plugin -y
```
If now work then Manual Install Docker Compose

**Plugin Folder**
```bash
mkdir -p ~/.docker/cli-plugins/
```
**Latest Compose Download**
```bash
curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 \
-o ~/.docker/cli-plugins/docker-compose
```

**Permission**
```bash
chmod +x ~/.docker/cli-plugins/docker-compose
```
**then heck Version**
```bash
docker compose version
```

<img width="1153" height="80" alt="image" src="https://github.com/user-attachments/assets/7b5e8ecd-d4d3-4ab9-bb97-74c4c997a7cd" />


### TASK 2 – First Compose File (Single Container)

**Step 1 – Create Folder**
```bash
mkdir compose-basics
cd compose-basics
```

**Step 2 – Create docker-compose.yml
vim docker-compose.yml

Paste:

version: '3.9'

services:
  nginx:
    image: nginx
    ports:
      - "8080:80"
🔹 Step 3 – Start
docker compose up

Open:

http://localhost:8080
🔹 Step 4 – Stop
docker compose down
🧠 What Happened Internally?

Compose automatically:

✔ Created network
✔ Created container
✔ Managed lifecycle

You didn’t manually create anything.
