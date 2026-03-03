
# Lab 3 – Docker Private Registry with Nginx Alpine
# Part 1

This lab demonstrates how to:
1. Run an **insecure Docker registry** locally.
2. Build a custom Docker image that installs and runs **Nginx** based on `alpine:latest`.
3. Push the image to the private registry.
4. Test pulling the image back from the registry.

---

## Step 1: Run an Insecure Registry
Start a local registry container:

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```
![](images/image1.png)

## Step 2: Allow Insecure Registry
By default, Docker requires HTTPS. Configure Docker to allow HTTP:

1. Create `/etc/docker/daemon.json` if it doesn’t exist:
   ```bash
   vim /etc/docker/daemon.json
   ```

2. Add:
   ```json
   {
     "insecure-registries": ["localhost:5000"]
   }
   ```

3. Restart Docker:
   ```bash
   systemctl restart docker
   ```
![](images/image2.png)

## Step 3: Create a Dockerfile for Nginx
Make a new directory and Dockerfile:

```bash
mkdir nginx-alpine && cd nginx-alpine
vim Dockerfile
```

Contents:

```dockerfile
FROM alpine:latest
RUN apk update && install nginx
CMD ["nginx", "-g", "daemon off;"]
```

**Explanation:**
- `CMD ["nginx", "-g", "daemon off;"]`: Run Nginx in foreground so Docker can manage it.

---

## Step 4: Build the Image
```bash
docker build -t localhost:5000/nginx-alpine .
```

- `-t localhost:5000/nginx-alpine`: Tags image with registry address and name.
- `.`: Build context is current directory.

---
![](images/image3.png)

## Step 5: Push the Image
```bash
docker push localhost:5000/nginx-alpine
```

This uploads the image to your local registry.

---

## Step 6: Test the Registry
1. List repositories:
   ```bash
   curl http://localhost:5000/v2/_catalog
   ```

2. Pull the image back:
   ```bash
   docker pull localhost:5000/nginx-alpine
   ```
![](images/image4.png)
---

# Part 2: WordPress + MySQL with Docker Compose

This lab demonstrates how to:
1. Run WordPress (`wordpress:latest`) with a MySQL (`mysql:5.7`) database using Docker Compose.
2. Persist MySQL data on the host machine.
3. Expose WordPress on port **8080**.

---

## Step 1: Create a Project Directory
```bash
mkdir wordpress-compose
cd wordpress-compose
```
![](images/image01.png)
---

## Step 2: Write the `docker-compose.yml`
# **VIP**
# mysql 5.7 version was incompatible with wordpress:latest
# wordpress couldn't access the database 
# when using mysql higher version as shown, wordpress accessed successfully

![](images/image02.png)


- **services:** → Top-level key that lists all containers.
  - **db (MySQL service):**
    - `image: mysql:5.7` → Pulls the official MySQL 5.7 image.
    - `environment:` → Sets required variables (e.g., `MYSQL_ROOT_PASSWORD`) so MySQL can start securely.
    - `volumes:` → Maps a host folder (`./mysql-data`) to `/var/lib/mysql` inside the container, ensuring database data persists.
  - **wordpress (WordPress service):**
    - `image: wordpress:latest` → Pulls the latest WordPress image.
    - `ports:` → Maps host port 8080 to container port 80, so WordPress is accessible at `http://localhost:8080`.
    - `environment:` → Provides DB connection details (host, user, password) so WordPress can talk to MySQL.
    - `depends_on:` → Ensures the database container starts before WordPress.


## Step 3: Start the Application
```bash
docker compose up -d
```

- `up`: Starts the services.
- `-d`: Detached mode (background).

![](images/image03.png)
Check running containers:
```bash
docker ps
```
![](images/image04.png)
---

## Step 4: Test the Setup
1. Open a browser on your host machine.
2. Navigate to:
   ```
   http://localhost:8080
   ```

![](images/image05.png)
---



