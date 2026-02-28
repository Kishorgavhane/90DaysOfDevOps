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

**Step 2 – Create docker-compose.yml**
```bash
vim docker-compose.yml
```

```bash
version: '3.9'

services:
  nginx:
    image: nginx
    ports:
      - "8080:80"
```

**Step 3 – Start**
```bash
docker compose up
```

<img width="1236" height="110" alt="image" src="https://github.com/user-attachments/assets/c832d8c4-627e-4669-970a-149e54c66b20" />


**Open: browser**
```text
http://localhost:8080
```

<img width="1236" height="299" alt="image" src="https://github.com/user-attachments/assets/d9c7e47b-0198-48d9-ada9-32a691074502" />


<img width="1246" height="604" alt="image" src="https://github.com/user-attachments/assets/7e6ca66b-6c15-4114-b872-bdadf926fb14" />


**Step 4 – Stop**
```bash
docker compose down
```

<img width="1246" height="106" alt="image" src="https://github.com/user-attachments/assets/b4087741-e386-4834-9a60-965acf86ea74" />


**Happened Internally?**

Compose automatically:

- Created network
- Created container
- Managed lifecycle

- **i didn’t manually create anything.**

### TASK 3 – Two Container Setup (WordPress + MySQL)

**Create New Folder**
```bash
mkdir wp-compose
cd wp-compose
```

**Create docker-compose.yml**
```bash
version: '3.9'

services:

  db:
    image: mysql:8
    container_name: wordpress-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress
    container_name: wordpress-app
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - db

volumes:
  db_data:
```

- **Concepts**
  
**Service Name = DNS Name**

`WORDPRESS_DB_HOST: db`

**Here:**
- `db` is service name
- Compose creates internal DNS
- WordPress connects using name, not IP

**Start Application**
```bash
docker compose up -d
```

<img width="1246" height="233" alt="image" src="https://github.com/user-attachments/assets/9bfb6d15-b54e-4599-9e04-123c4384a84a" />


**Open:**
```text
http://localhost:8080
```

<img width="1293" height="461" alt="image" src="https://github.com/user-attachments/assets/8927fdf5-6cd6-41f2-bcf6-56987743094c" />


> I will see WordPress setup page.

**Test Persistence**
```bash
docker compose down
docker compose up -d
```

<img width="1190" height="259" alt="image" src="https://github.com/user-attachments/assets/e25df956-2fcc-4451-bdef-c00709758560" />


> Is WordPress data still there?

<img width="1293" height="461" alt="image" src="https://github.com/user-attachments/assets/6d3d549d-806d-4735-8dd7-212d317ecb72" />


- Yes → because of named volume `db_data`


### TASK 4 – Important Compose Commands

**Detached Mode**
```bash
docker compose up -d
```
**View Running Services**
```bash
docker compose ps
```
**View Logs (All)**
```bash
docker compose logs -f
```
**View Logs (Specific)**
```bash
docker compose logs -f wordpress
```
**Stop Without Removing**
```bash
docker compose stop
```
**Remove Everything**
```bash
docker compose down
```
**Rebuild**
```bash
docker compose up -d --build
```
