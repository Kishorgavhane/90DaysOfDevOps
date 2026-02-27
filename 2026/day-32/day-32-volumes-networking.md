# Day 32 – Docker Volumes & Networking

### TASK 1 – The Problem (Ephemeral Containers)

- **Step 1 –I Run MySQL (Without Volume)**

```bash
docker run -d --name mysql-test \
-e MYSQL_ROOT_PASSWORD=1234 \
-e MYSQL_DATABASE=testdb \
-p 3306:3306 mysql:8
```

- **Step 2 – Enter Container**

```bash
docker exec -it mysql-test mysql -u root -p
```
