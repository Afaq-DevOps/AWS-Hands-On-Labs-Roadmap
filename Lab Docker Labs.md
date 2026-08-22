# Lab 01 — Run Your First Docker Container Locally

## Objective

In this lab, students will:

- Verify Docker installation
- Pull a Docker image
- Run one container
- Understand image vs container
- Understand port mapping
- View logs
- Enter a running container
- Stop, start, and remove a container

## Architecture

```
Browser
   |
localhost:8080
   |
Docker Engine
   |
Nginx Container
   |
Port 80
```

## Step 1: Verify Docker

Run:

```
docker --version
```

Check Docker Compose:

```
docker compose version
```

Test Docker:

```
docker run hello-world
```

If Docker works, you should see a success message.

## Step 2: Understand the Basic Docker Flow

Docker normally works like this:

```
Docker Registry
      |
      | docker pull
      v
Docker Image
      |
      | docker run
      v
Container
```

An **image** is a template.

A **container** is a running instance of that image.

Example:

```
nginx:alpine
    |
   Image
    |
docker run
    |
    v
metapi-web
 Container
```

## Step 3: Pull the Nginx Image

Run:

```
docker pull nginx:alpine
```

Check downloaded images:

```
docker images
```

You should see something similar to:

```
REPOSITORY   TAG
nginx        alpine
```

## Step 4: Run the Nginx Container

Run:

```
docker run -d --name metapi-web -p 8080:80 nginx:alpine
```

Explanation:

```
docker run
```

Creates and starts a container.

```
-d
```

Runs it in the background.

```
--name metapi-web
```

Sets the container name.

```
-p 8080:80
```

Maps:

```
Laptop Port 8080
        |
        v
Container Port 80
```

## Step 5: Check the Running Container

Run:

```
docker ps
```

You should see:

```
metapi-web
```

## Step 6: Open the Website

Open:

```
http://localhost:8080
```

You should see:

```
Welcome to nginx!
```

## Step 7: Understand Port Mapping

Inside the container, Nginx listens on port:

```
80
```

But on your laptop you access it using:

```
8080
```

Flow:

```
Browser
   |
localhost:8080
   |
Docker Port Mapping
   |
Container:80
   |
Nginx
```

## Step 8: View Container Logs

Run:

```
docker logs metapi-web
```

Refresh the website.

Run again:

```
docker logs metapi-web
```

You should see HTTP requests.

## Step 9: Enter the Container

Run:

```
docker exec -it metapi-web sh
```

Now you are inside the container.

Run:

```
hostname
```

Check the Nginx web directory:

```
ls /usr/share/nginx/html
```

Exit:

```
exit
```

## Step 10: Create Your Own Website

Create:

```
index.html
```

Add:

```
<!DOCTYPE html>
<html>
<head>
    <title>MetaPi Docker Lab 01</title>
</head>
<body>

<h1>MetaPi Docker Training</h1>

<h2>Lab 01</h2>

<p>This website is running inside a Docker container.</p>

</body>
</html>
```

Copy it into the container:

```
docker cp index.html metapi-web:/usr/share/nginx/html/index.html
```

Refresh:

```
http://localhost:8080
```

## Step 11: Stop the Container

```
docker stop metapi-web
```

Check:

```
docker ps
```

The container should no longer be running.

## Step 12: Start It Again

```
docker start metapi-web
```

Open:

```
http://localhost:8080
```

## Step 13: Remove the Container

Stop it:

```
docker stop metapi-web
```

Remove it:

```
docker rm metapi-web
```

Check images:

```
docker images
```

The Nginx image should still exist.

This teaches an important concept:

> Removing a container does not automatically remove its image.

## Lab 01 Completed

```
Image
  |
docker run
  |
Container
  |
Port Mapping
  |
Browser
```

---

# Lab 02 — Build and Run Multiple Docker Containers Locally

This lab is the most important one for understanding **Docker concepts**.

Students will manually build and connect several containers before learning Docker Compose.

## Objective

Students will learn:

- Dockerfile
- Image building
- Container lifecycle
- Docker networks
- Container DNS
- Environment variables
- Multiple containers
- Docker volumes
- Persistent data
- Container-to-container communication
- Why containers should be separated by responsibility

## Application Architecture

We will run:

```
Browser
   |
   +----------------------+
   |                      |
localhost:8081      localhost:8082
   |                      |
App Container 1      App Container 2
   \                      /
    \                    /
       Docker Network
             |
         Redis Container
             |
         Docker Volume
```

The two application containers will use the same Redis container.

## Docker Concept 1 — One Container, One Main Responsibility

A common container design principle is:

```
App Container
    |
Runs Application

Redis Container
    |
Runs Redis
```

We do not install everything inside one giant container.

Instead:

```
Application
    +
Redis
```

becomes:

```
App Container
      |
Docker Network
      |
Redis Container
```

This makes applications easier to:

- Replace
- Scale
- Update
- Debug
- Manage

## Step 1: Create the Project Directory

Create:

```
metapi-docker-lab02
```

Enter it:

```
cd metapi-docker-lab02
```

## Step 2: Create the Application

Create:

```
app.py
```

Add:

```
from flask import Flask
import redis
import os
import socket

app = Flask(__name__)

redis_host = os.getenv("REDIS_HOST", "redis")

r = redis.Redis(
    host=redis_host,
    port=6379,
    decode_responses=True
)

@app.route("/")
def home():
    count = r.incr("visits")

    return f"""
    <html>
    <body>

    <h1>MetaPi Docker Lab 02</h1>

    <p>Container Hostname: {socket.gethostname()}</p>

    <p>Total Visits: {count}</p>

    <p>Redis Host: {redis_host}</p>

    </body>
    </html>
    """

app.run(
    host="0.0.0.0",
    port=5000
)
```

## Step 3: Create `requirements.txt`

Create:

```
requirements.txt
```

Add:

```
flask
redis
```

## Docker Concept 2 — Dockerfile

A Dockerfile tells Docker how to build an image.

Create:

```
Dockerfile
```

Add:

```
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

## What Each Dockerfile Instruction Means

```
FROM python:3.12-slim
```

Start from a Python base image.

```
WORKDIR /app
```

Set `/app` as the working directory.

```
COPY requirements.txt .
```

Copy the dependency file.

```
RUN pip install ...
```

Install Python packages while building the image.

```
COPY app.py .
```

Copy the application code.

```
EXPOSE 5000
```

Document that the application listens on port 5000.

```
CMD ["python", "app.py"]
```

Start the application when the container starts.

## Docker Concept 3 — Build an Image

Run:

```
docker build -t metapi-counter:v1 .
```

Check:

```
docker images
```

You should see:

```
metapi-counter   v1
```

The flow is:

```
Application Code
      |
Dockerfile
      |
docker build
      |
Docker Image
```

## Docker Concept 4 — Docker Network

Containers are isolated by default.

We will create a custom network so our containers can communicate.

Run:

```
docker network create metapi-network
```

Check:

```
docker network ls
```

Architecture:

```
App Container
      |
metapi-network
      |
Redis Container
```

## Why Use a Docker Network?

Without a network, applications should not rely on random container IP addresses.

Docker networking gives us:

```
Container Name
      |
Docker DNS
      |
Container IP
```

So the application can contact:

```
metapi-redis
```

instead of:

```
172.18.0.5
```

## Docker Concept 5 — Container DNS

Docker provides internal DNS on user-defined networks.

If Redis container is named:

```
metapi-redis
```

the application can connect using:

```
metapi-redis:6379
```

Docker resolves the name automatically.

This is very important later when students learn container orchestration.

## Docker Concept 6 — Docker Volume

Containers should be treated as temporary.

If a container is deleted:

```
Container
   |
Deleted
```

data stored only inside it may disappear.

A Docker Volume stores important data outside the container lifecycle.

Create:

```
docker volume create metapi-redis-data
```

Check:

```
docker volume ls
```

Architecture:

```
Redis Container
      |
Docker Volume
      |
Persistent Data
```

## Step 4: Start Redis

Run:

```
docker run -d --name metapi-redis --network metapi-network -v metapi-redis-data:/data redis:7-alpine
```

Check:

```
docker ps
```

## Understand This Command

```
--name metapi-redis
```

Container name.

```
--network metapi-network
```

Connect it to our custom network.

```
-v metapi-redis-data:/data
```

Attach the volume.

```
redis:7-alpine
```

Image used to create the container.

## Docker Concept 7 — Environment Variables

Our Flask application needs to know where Redis is.

Instead of hard-coding the address into the image, we pass:

```
REDIS_HOST
```

as an environment variable.

This makes the same image reusable.

## Step 5: Run Application Container 1

```
docker run -d --name metapi-app-1 --network metapi-network -e REDIS_HOST=metapi-redis -p 8081:5000 metapi-counter:v1
```

Open:

```
http://localhost:8081
```

You should see:

```
MetaPi Docker Lab 02

Container Hostname: <container-id>

Total Visits: 1
```

Refresh several times.

The counter increases.

## Step 6: Run Application Container 2

Run another container from the same image:

```
docker run -d --name metapi-app-2 --network metapi-network -e REDIS_HOST=metapi-redis -p 8082:5000 metapi-counter:v1
```

Open:

```
http://localhost:8082
```

Notice:

```
Container Hostname
```

is different.

But the visit counter continues.

Why?

Because both containers use the same Redis service.

```
App 1 -----\
            \
             Redis
            /
App 2 -----/
```

## Docker Concept 8 — One Image, Multiple Containers

Both applications came from:

```
metapi-counter:v1
```

But they are separate containers.

```
               metapi-counter:v1
                       |
              ------------------
              |                |
          App-1             App-2
```

This is one of the most important container concepts.

> One image can create many containers.

## Docker Concept 9 — Containers Are Isolated

App-1 has its own:

- Processes
- Filesystem layer
- Hostname

App-2 has its own:

- Processes
- Filesystem layer
- Hostname

But they can communicate through the Docker network.

## Step 7: Inspect the Network

Run:

```
docker network inspect metapi-network
```

You should see:

```
metapi-app-1
metapi-app-2
metapi-redis
```

## Step 8: Test Docker DNS

Run:

```
docker exec metapi-app-1 python -c "import socket; print(socket.gethostbyname('metapi-redis'))"
```

Docker should return the Redis container's internal IP.

The important point is:

> The application does not need to know the IP address.

It only needs the service/container name.

## Docker Concept 10 — Internal vs Published Ports

The Flask application listens on:

```
5000
```

inside both containers.

But externally:

```
App-1
5000 → 8081

App-2
5000 → 8082
```

So:

```
localhost:8081
```

and:

```
localhost:8082
```

reach two different containers.

Redis listens on:

```
6379
```

but we did not publish it to the laptop.

Only other containers on the network can use it.

This is better than exposing Redis publicly.

## Step 9: Test Persistence

Check the current visitor count.

Stop Redis:

```
docker stop metapi-redis
```

Remove Redis:

```
docker rm metapi-redis
```

Create it again using the same volume:

```
docker run -d --name metapi-redis --network metapi-network -v metapi-redis-data:/data redis:7-alpine
```

Refresh:

```
http://localhost:8081
```

If Redis persisted the data to its volume as expected, the counter continues instead of starting from zero.

## Docker Concept 11 — Container vs Volume Lifecycle

```
Container
   |
Can be deleted
   |
Recreated
```

But:

```
Docker Volume
     |
Remains
     |
Persistent Data
```

This is why persistent application data should not depend only on a container's writable layer.

## Step 10: View Logs

App 1:

```
docker logs metapi-app-1
```

App 2:

```
docker logs metapi-app-2
```

Redis:

```
docker logs metapi-redis
```

## Step 11: Enter a Container

```
docker exec -it metapi-app-1 sh
```

Inside:

```
hostname
```

Exit:

```
exit
```

## Step 12: Lab Verification

Students should now understand:

```
Dockerfile
     |
Build
     |
Image
     |
Run
     |
Multiple Containers
     |
Docker Network
     |
Container DNS
     |
Redis
     |
Volume
```

## Step 13: Clean Up

Remove application containers:

```
docker rm -f metapi-app-1
docker rm -f metapi-app-2
docker rm -f metapi-redis
```

Remove network:

```
docker network rm metapi-network
```

Remove volume if you do not need the data:

```
docker volume rm metapi-redis-data
```

---

# Lab 03 — Run a Multi-Container Application Using Docker Compose

In Lab 02, we manually created:

```
Network
Volume
Redis Container
App Container 1
App Container 2
Environment Variables
Port Mappings
```

This is manageable for three containers.

But imagine:

```
20 Containers
```

Running 20 long `docker run` commands becomes difficult.

Docker Compose solves this problem.

## Objective

Students will learn:

- Docker Compose
- YAML structure
- Services
- Networks
- Volumes
- Environment variables
- Service discovery
- Dependencies
- Health checks
- Multi-container lifecycle

## Architecture

```
Browser
   |
localhost:8080
   |
Nginx
   |
Flask App
   |
Redis
   |
Docker Volume
```

Three containers will run together.

## Step 1: Create Project

Create:

```
metapi-docker-lab03
```

Enter:

```
cd metapi-docker-lab03
```

## Step 2: Create `app.py`

```
from flask import Flask
import redis
import os
import socket

app = Flask(__name__)

redis_host = os.getenv("REDIS_HOST", "redis")

r = redis.Redis(
    host=redis_host,
    port=6379,
    decode_responses=True
)

@app.route("/")
def home():

    count = r.incr("visits")

    return f"""
    <html>
    <body>

    <h1>MetaPi Docker Compose Lab</h1>

    <p>Application Container: {socket.gethostname()}</p>

    <p>Total Visits: {count}</p>

    <p>Frontend Proxy: Nginx</p>

    <p>Backend: Flask</p>

    <p>Data Store: Redis</p>

    </body>
    </html>
    """

app.run(
    host="0.0.0.0",
    port=5000
)
```

## Step 3: Create `requirements.txt`

```
flask
redis
```

## Step 4: Create Dockerfile

```
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

## Step 5: Create Nginx Configuration

Create:

```
nginx.conf
```

Add:

```
events {}

http {

    resolver 127.0.0.11 valid=5s;

    upstream backend {
        zone backend 64k;
        server app:5000 resolve;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;

            proxy_http_version 1.1;

            proxy_set_header Connection "";

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

Notice:

```
app:5000
```

Nginx does not need the Flask container IP.

Docker Compose provides service discovery using the service name:

```
app
```

## Step 6: Create `compose.yaml`

```
services:

  redis:
    image: redis:7-alpine

    volumes:
      - redis-data:/data

    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5


  app:
    build: .

    environment:
      REDIS_HOST: redis

    depends_on:
      redis:
        condition: service_healthy


  nginx:
    image: nginx:alpine

    ports:
      - "8080:80"

    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro

    depends_on:
      - app


volumes:

  redis-data:
```

Docker Compose automatically creates a default network for these services, so for this beginner lab we do not even need to manually define one.

## Step 7: Understand Compose Services

We defined:

```
redis
app
nginx
```

Docker Compose creates separate containers for each service.

```
compose.yaml
     |
     +---- Redis Container
     |
     +---- Flask Container
     |
     +---- Nginx Container
```

## Step 8: Validate the Compose File

Run:

```
docker compose config
```

If the file is valid, continue.

## Step 9: Start Everything

Run:

```
docker compose up -d --build
```

Docker Compose will:

```
Build Flask Image
        |
Pull Redis Image
        |
Pull Nginx Image
        |
Create Network
        |
Create Volume
        |
Start Redis
        |
Start Flask
        |
Start Nginx
```

## Step 10: Check Running Services

```
docker compose ps
```

You should see:

```
redis
app
nginx
```

## Step 11: Open the Application

Open:

```
http://localhost:8080
```

Request flow:

```
Browser
   |
localhost:8080
   |
Nginx
   |
app:5000
   |
Flask
   |
redis:6379
   |
Redis
```

## Step 12: View Logs

All services:

```
docker compose logs
```

Follow logs:

```
docker compose logs -f
```

Only application:

```
docker compose logs app
```

Only Nginx:

```
docker compose logs nginx
```

Only Redis:

```
docker compose logs redis
```

## Step 13: Enter the Flask Container

```
docker compose exec app sh
```

Run:

```
hostname
```

Exit:

```
exit
```

## Step 14: Test Service Discovery

```
docker compose exec app python -c "import socket; print(socket.gethostbyname('redis'))"
```

Docker resolves:

```
redis
```

to the Redis container.

## Step 15: Restart Everything

```
docker compose restart
```

Refresh:

```
http://localhost:8080
```

The Redis counter should continue because data is stored in the named volume.

## Step 15: Scale app

```
docker compose up -d --build --scale app=4
```

chala rahe ho, aur Nginx config mein ye hai:
# Simple answer

Architecture kuch aisi ho jati hai:

```
Browser
   |
localhost:8080
   |
Nginx
   |
-------------------------
|         |         |    |
App-1    App-2    App-3 App-4
   \        |        |    /
              Redis
```

To **haan, conceptually Nginx yahan load balancing kar raha hota hai**.

## Step 16: Stop Without Removing Containers

```
docker compose stop
```

Start again:

```
docker compose start
```

## Step 18: Remove Containers

```
docker compose down
```

This removes containers and the Compose network.

The named volume normally remains.

## Step 19: Remove Containers and Persistent Data

```
docker compose down -v
```

Now the Redis volume is also deleted.

---

# Final Learning Progression

```
LAB 01

One Container
     |
Image
Container
Ports
Logs
Exec
Lifecycle

        ↓

LAB 02

Multiple Containers
     |
Dockerfile
Build
Network
DNS
Environment Variables
Volumes
Persistence
Container Isolation

        ↓

LAB 03

Docker Compose
     |
Services
Automatic Networking
Volumes
Dependencies
Health Checks
Multi-Container Management
```

The most important lesson for students after these three labs should be:

> **A container is one isolated running application. A Docker network allows containers to communicate. A Docker volume keeps data persistent. Docker Compose allows us to define and manage multiple containers as one application.**