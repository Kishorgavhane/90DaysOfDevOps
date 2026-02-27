# Day 32 – Docker Volumes & Networking

### TASK 1 – The Problem (Ephemeral Containers)

- **Step 1 –I Run MySQL (Without Volume)**

```bash
docker run -d --name mysql-test \
-e MYSQL_ROOT_PASSWORD=1234 \
-e MYSQL_DATABASE=testdb \
-p 3306:3306 mysql:8
```

<img width="1071" height="97" alt="image" src="https://github.com/user-attachments/assets/2992a662-226d-45e9-a584-ee885f1adb06" />



- **Step 2 – Enter Container**

```bash
docker exec -it mysql-test mysql -u root -p
```
**password: `1234`**

<img width="1071" height="97" alt="image" src="https://github.com/user-attachments/assets/6478e04c-74cc-42eb-be00-16b2286cd0d9" />



**Check Databases**
```bash
SHOW DATABASES;
```

<img width="1070" height="198" alt="image" src="https://github.com/user-attachments/assets/bea3fdf5-e733-41c2-8f8c-7dd6678f8937" />


**Select Database**
```bash
USE testdb;
```

<img width="1070" height="48" alt="image" src="https://github.com/user-attachments/assets/682c724c-551e-4e70-ada3-64a9fd75a608" />


**Now i create table:**

```bash
CREATE TABLE users (id INT, name VARCHAR(50));
INSERT INTO users VALUES (1, 'Kishor');
```

<img width="1099" height="96" alt="image" src="https://github.com/user-attachments/assets/75673cec-ff68-48f3-b120-60a657745d35" />


**Verify Data**
```bash
SELECT * FROM users;
```

<img width="1098" height="126" alt="image" src="https://github.com/user-attachments/assets/6647f5d7-dddd-4b9f-9493-3c010c716394" />


**Error Happened?**

MySQL requires:
- Select database
- Then create table

Without USE database_name; → MySQL doesn’t know where to create table.

### Step 3 – now Stop & Remove Container

```bash
docker stop mysql-test
docker rm mysql-test
```

<img width="1117" height="190" alt="image" src="https://github.com/user-attachments/assets/d172b6b6-4206-4983-8eff-6b51afb58a95" />


### Step 4 – Run New Container Again (Same Command)

```bash
docker run -d --name mysql-test \
-e MYSQL_ROOT_PASSWORD=1234 \
-e MYSQL_DATABASE=testdb \
-p 3306:3306 mysql:8
```

<img width="1115" height="616" alt="image" src="https://github.com/user-attachments/assets/316172b7-1856-44a1-9ae8-2d3be1af87fb" />


- **Data is gone**
**why**
> Because containers are ephemeral. Data stored inside container filesystem → Deleted when container removed.

### TASK 2 – Named Volumes (Permanent Data)

**Step 1 – Create Volume**
```bash
docker volume create mysql-data
```
**Check:**
```bash
docker volume ls
```

<img width="1117" height="279" alt="image" src="https://github.com/user-attachments/assets/885cdcc5-5bd5-4cd3-975c-1ec0b7d261a0" />


**Step 2 – Run MySQL With Volume**
```bash
docker run -d --name mysql-test \
-e MYSQL_ROOT_PASSWORD=1234 \
-e MYSQL_DATABASE=testdb \
-v mysql-data:/var/lib/mysql \
-p 3306:3306 mysql:8
```
**Important:**
```bash
-v mysql-data:/var/lib/mysql
```
> This stores DB data in volume.

<img width="1117" height="124" alt="image" src="https://github.com/user-attachments/assets/a047f157-5028-40ea-9536-9552ab92b333" />

**Step 3 – Add Data Again**
```bash
CREATE TABLE users (id INT, name VARCHAR(50));
INSERT INTO users VALUES (1, 'Kishor');
```

<img width="1117" height="554" alt="image" src="https://github.com/user-attachments/assets/1b796ad1-0595-4437-b410-371e4e8d415f" />


**Verify Data**
```bash
SELECT * FROM users;
```
**Step 4 – then i Remove Container**
```bash
docker stop mysql-test
docker rm mysql-test
```
**Step 5 – Run New Container With SAME Volume**

```bash
docker run -d --name mysql-test \
-e MYSQL_ROOT_PASSWORD=1234 \
-e MYSQL_DATABASE=testdb \
-v mysql-data:/var/lib/mysql \
-p 3306:3306 mysql:8
```

<img width="1130" height="638" alt="image" src="https://github.com/user-attachments/assets/79c9fdaf-07fd-4171-8ac6-8a50d54f140e" />


**Why?**

Because volume lives outside container.
- Container deleted
- Volume survives.

### TASK 3 – Bind Mount

**Step 1 – Create Host Folder**
```bash
mkdir ~/mywebsite
echo "<h1>Hello from Host </h1>" > ~/mywebsite/index.html
```

<img width="1130" height="354" alt="image" src="https://github.com/user-attachments/assets/1d02db3b-fe13-4d95-9fcb-afb222e2d979" />

**Step 2 – Run Nginx With Bind Mount**
```bash
docker run -d --name nginx-test \
-p 8080:80 \
-v ~/mywebsite:/usr/share/nginx/html \
nginx
```
**Output**

<img width="1170" height="169" alt="image" src="https://github.com/user-attachments/assets/415ee7ee-ceaf-4c19-87da-ce73a0f84cb5" />


**Step 3 – Then I Edit File on Host**
```bash
echo "<h1>Updated Live </h1>" > ~/mywebsite/index.html
```

**Output**

<img width="1170" height="169" alt="image" src="https://github.com/user-attachments/assets/1e243aa0-092b-4131-8b4c-2050dea585d4" />

**repeted I Edit File on Host**
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kishor Gavhane | DevOps Portfolio</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #0f172a;
            color: #f1f5f9;
            margin: 0;
            padding: 40px;
        }
        h1, h2 {
            color: #38bdf8;
        }
        section {
            margin-bottom: 30px;
        }
        ul {
            line-height: 1.8;
        }
        a {
            color: #22d3ee;
            text-decoration: none;
        }
    </style>
</head>
<body>

    <h1>Hi, I'm Kishor Gavhane 👋</h1>
    <p>DevOps & Cloud Engineer with 7+ years of IT experience in infrastructure, cloud, and automation.</p>

    <section>
        <h2>🔧 Skills</h2>
        <ul>
            <li>Azure & AWS DevOps</li>
            <li>AWS (Fundamentals)</li>
            <li>Linux (Ubuntu)</li>
            <li>CI/CD Pipelines</li>
            <li>Docker & Kubernetes (Basics)</li>
            <li>Terraform (Basics)</li>
            <li>Networking & Firewalls</li>
        </ul>
    </section>

    <section>
        <h2>🚀 What I'm Learning</h2>
        <ul>
            <li>Advanced CI/CD</li>
            <li>Kubernetes</li>
            <li>Terraform</li>
            <li>AWS</li>
        </ul>
    </section>

    <section>
        <h2>📫 Connect with Me</h2>
        <ul>
            <li>Email: kishorgavhane.dev@gmail.com</li>
            <li>LinkedIn: <a href="https://www.linkedin.com/in/kishor-g-dev" target="_blank">www.linkedin.com/in/kishor-g-dev</a></li>
        </ul>
    </section>

    <section>
        <h2>🏅 Certifications</h2>
        <ul>
            <li>Microsoft Azure Administrator (AZ-104)</li>
        </ul>
    </section>

</body>
</html>
```

**Output**

<img width="1232" height="681" alt="image" src="https://github.com/user-attachments/assets/41e41a5a-0cbd-46f5-8181-4a8a9a4f214c" />


**Difference**

| Named Volume          | Bind Mount           |
| --------------------- | -------------------- |
| Managed by Docker     | Managed by host      |
| Stored in Docker area | Direct host folder   |
| Good for DB           | Good for development |


### TASK 4 – Docker Networking Basics

**List Networks**
```bash
docker network ls
```

will see:

- bridge
- host
- none

**Inspect Bridge**
```bash
docker network inspect bridge
```

**Run Two Containers**
```nash
docker run -dit --name c1 ubuntu
docker run -dit --name c2 ubuntu
```

**Enter c1:**
```bash
docker exec -it c1 bash
```

<img width="1153" height="307" alt="image" src="https://github.com/user-attachments/assets/63396810-58c8-4a24-a2a6-d8812ab6f460" />


**Try:**
```bash
ping c2
```
> Won’t work.

<img width="1153" height="148" alt="image" src="https://github.com/user-attachments/assets/1aa20ecd-0805-4920-a280-cd49004f5a42" />


**Now try using IP:**
```bash
ping <c2_ip>
```
> Works.

<img width="1153" height="314" alt="image" src="https://github.com/user-attachments/assets/83e93038-8cc3-4e8f-acd8-35aaac8d65aa" />


### TASK 5 – Custom Network

**Create Network**
```bash
docker network create my-app-net
```

<img width="1153" height="103" alt="image" src="https://github.com/user-attachments/assets/ba90ff48-5ebe-43e4-8204-e3fedc870b6e" />


**Run Containers On It**
```bash
docker run -dit --name c1 --network my-app-net ubuntu
docker run -dit --name c2 --network my-app-net ubuntu
```

<img width="1153" height="103" alt="image" src="https://github.com/user-attachments/assets/ac1bb7fb-691f-46ca-8b09-2dc2bbe35256" />


**Enter c1:**
```bash
docker exec -it c1 bash
ping c2
```

<img width="1153" height="492" alt="image" src="https://github.com/user-attachments/assets/42d1f5b7-3243-468e-857d-70eb0a3862ac" />


- Now works by name.

**Why?**

> Custom bridge enables automatic DNS resolution.Default bridge does not support name-based communication easily.

### TASK 6 – Put It Together

**Create Network**
```bash
docker network create app-network
```

<img width="1153" height="44" alt="image" src="https://github.com/user-attachments/assets/77cc526a-e753-4ac6-9878-cc43f0a6480c" />


**Run MySQL With Volume**
```bash
docker run -d --name mysql-db \
--network app-network \
-e MYSQL_ROOT_PASSWORD=1234 \
-v mysql-data:/var/lib/mysql \
mysql:8
```

<img width="1153" height="344" alt="image" src="https://github.com/user-attachments/assets/e8795e52-635d-4bd5-9a6a-b63904f87c87" />


**Run App Container**
```bash
docker run -dit --name app \
--network app-network \
ubuntu
```

<img width="1153" height="74" alt="image" src="https://github.com/user-attachments/assets/2c532cf6-e341-4954-a73e-a525da421025" />


**Enter app:**
```bash
docker exec -it app bash
ping mysql-db
```

<img width="1153" height="184" alt="image" src="https://github.com/user-attachments/assets/f5bbf95e-c4b1-4c2d-899c-9e88745a1ebe" />


- Container communicates by name.

