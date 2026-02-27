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
