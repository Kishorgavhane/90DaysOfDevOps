# 🚀 DAY 31 – Dockerfile Deep Dive

## TASK 1 – Your First Dockerfile

- **Create Project Folder**

```bash
mkdir my-first-image
cd my-first-image
```

- **Create Dockerfile**
```bash
vim Dockerfile
```
- **Demo Ex.:**
```bash
FROM ubuntu

RUN apt update && apt install -y curl

CMD ["echo", "Hello from my custom image!"]
```


<img width="1086" height="185" alt="image" src="https://github.com/user-attachments/assets/2ede79ca-7c7b-4a5e-ba63-6eed7978196e" />



**Each Line Explanation**

**FROM ubuntu**

- Base image
- Every Dockerfile must start with FROM
- It defines foundation layer

**RUN apt update && apt install -y curl**

- Executes command during build
- Creates a new image layer
- Installs curl inside image

**CMD ["echo", "..."]**

- Default command when container runs
- Can be overridden

**Build Image**
```bash
docker build -t my-ubuntu:v1 .
```
**Important:**
> `.` means current directory (build context).

**Run Image**
```bash
docker run my-ubuntu:v1
```

**see:**
```bash
Hello from my custom image!
```

<img width="1086" height="185" alt="image" src="https://github.com/user-attachments/assets/dcdf92c3-c98f-4468-b9b9-99bbf2dd458a" />



---

## TASK 2 – Understand All Dockerfile Instructions

- Created folder:
```bash
mkdir dockerfile-demo
cd dockerfile-demo
```

- Create file:
```bash
FROM ubuntu

RUN apt update && apt install -y nginx

WORKDIR /app

COPY . .

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

- **Each Instruction Does**

| Instruction | Purpose                       |
| ----------- | ----------------------------- |
| FROM        | Base image                    |
| RUN         | Execute build-time commands   |
| WORKDIR     | Set working directory         |
| COPY        | Copy files from host to image |
| EXPOSE      | Document port (not publish)   |
| CMD         | Default runtime command       |


**output**

<img width="1086" height="449" alt="image" src="https://github.com/user-attachments/assets/cdfe733d-a950-4036-ae97-4463c0e16e6e" />

---

## TASK 3 – CMD vs ENTRYPOINT (VERY IMPORTANT)

- **Case 1 – CMD**

> Dockerfile:
```bash
FROM ubuntu
CMD ["echo", "hello"]
```
- **Build & Run:**
```bash
docker build -t cmd-test .
docker run cmd-test
```

- **Output:**
```text
hello
```
- **Now override:**
```bash
docker run cmd-test date
```
- **Output:**
```text
Current date
```
- **CMD can be overridden.**


<img width="1186" height="520" alt="image" src="https://github.com/user-attachments/assets/5206bf00-64c0-467c-b49d-d1c0e1eddc3f" />


### Case 2 – ENTRYPOINT

- **Dockerfile:**
```bash
FROM ubuntu
ENTRYPOINT ["echo"]
```
- **Build & Run:**
```bash
docker build -t entry-test .
docker run entry-test hello
```
- **Output:**
```text
hello
```
***Here:**
> ENTRYPOINT cannot be overridden easily.


<img width="1124" height="315" alt="image" src="https://github.com/user-attachments/assets/954c6654-cdb7-490b-9d4e-f1d922cb521b" />



### When To Use What?

| CMD             | ENTRYPOINT       |
| --------------- | ---------------- |
| Default command | Fixed executable |
| Can override    | Hard to override |
| Flexible        | Strict           |


---

## TASK 4 – Build Simple Web App Image

- **Create index.html**
```bash
mkdir my-website
cd my-website
vim index.html
```

- **Add:**
```text
<h1>Hello Kishor's Docker Website 🚀</h1>
```
- **Dockerfile**
```bash
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/

EXPOSE 80
```

- **Build**
```bash
docker build -t my-website:v1 .
```

<img width="1028" height="130" alt="image" src="https://github.com/user-attachments/assets/9c6b225f-8a79-4621-a970-bd1defef45cf" />



- **Run**
```
docker run -d -p 8080:80 my-website:v1
```

<img width="1177" height="130" alt="image" src="https://github.com/user-attachments/assets/21aa38dd-d4f8-4abd-ab4b-27abf00a66a8" />


**I deployed my own containerized website.**

---

## ASK 5 – .dockerignore

- Create file:
```bash
vim .dockerignore
```
- Add:
```bash
node_modules
.git
*.md
.env
```

**Why Important?**

Without .dockerignore:
- Image size increases
- Sensitive files copied
- Build slower
It works like .gitignore.

---

## TASK 6 – Build Optimization

- Example Bad Order
```bash
COPY . .
RUN npm install
```
> If any file changes → npm install runs again.

- Optimized Version
```bash
COPY package.json .
RUN npm install
COPY . .
```

**Now:**
> If only app code changes → npm install layer reused.

**Why Layer Order Matters?**

> Docker caches layer by layer.
> If early layer changes → all layers rebuild.


---

## HIGHLIGHT POINTS

- Dockerfile = Recipe for building images
- FROM is mandatory
- RUN creates layers
- CMD can override
- ENTRYPOINT is fixed
- .dockerignore improves security & speed
- Layer order impacts build performance
- nginx:alpine = lightweight production base

---
