# DOCKER COMPOSE

## Docker Compose

* Docker Compose is a tool used to define and run multi-container Docker applications.
* Instead of running multiple `docker run` commands manually, we can define all containers in a single YAML file and start everything together.

---

# Why Docker Compose?

Docker Compose helps us manage:

* Multiple Containers
* Networks
* Volumes
* Environment Variables

using a single YAML file.

---

# YAML

YAML = Yet Another Markup Language

## Features of YAML

* Human readable
* Indentation based
* Used for configuration files

---

# Default Docker Compose File Names

```bash
docker-compose.yaml
```

```bash
docker-compose.yml
```

```bash
compose.yaml
```

```bash
compose.yml
```

---

# Why Docker Compose is Important

Suppose your application needs:

* Spring Boot Application
* MongoDB Database
* Network Connection
* Volume for Persistent Storage

## Without Docker Compose

* Need multiple `docker run` commands
* Difficult to manage
* Difficult to maintain

## With Docker Compose

* Everything inside one YAML file
* Easy to start
* Easy to stop
* Easy to manage

---

# Real-Time Example

## Spring Boot Application + MongoDB

---

# Docker Run Commands

## Spring Boot Application Container

```bash
docker run -d --name springapp \
  --network jionetwork \
  -p 8080:8080 \
  -e MONGO_DB_HOSTNAME=mongo \
  -e MONGO_DB_USERNAME=devdb \
  -e MONGO_DB_PASSWORD=dev@123 \
  springimage
```

---

## MongoDB Container

```bash
docker run -d --name mongo \
  -v jiovolume:/data/db \
  --network jionetwork \
  -e MONGO_INITDB_ROOT_USERNAME=devdb \
  -e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
  mongo:6.0
```

---

# Interview Question

## App or DB — Which Should Stop First?

### Answer

First stop:

* Application Container

Then stop:

* Database Container

## Reason

* Application depends on database.
* Helps avoid connection failure errors.

---

# Structure of Docker Compose File

```yaml
version:
services:
volumes:
networks:
```

---

# Docker Compose Example

## docker-compose.yaml

```yaml
version: "3"

services:

  springapp:
    image: springimage

    ports:
      - "8080:8080"

    depends_on:
      - mongo

    networks:
      - jionetwork

    environment:
      - MONGO_DB_HOSTNAME=mongo
      - MONGO_DB_USERNAME=devdb
      - MONGO_DB_PASSWORD=dev@123

  mongo:
    image: mongo:6.0

    networks:
      - jionetwork

    environment:
      - MONGO_INITDB_ROOT_USERNAME=devdb
      - MONGO_INITDB_ROOT_PASSWORD=dev@123

    volumes:
      - jiovolume:/data/db

networks:
  jionetwork:
    name: jionetwork
    driver: bridge

volumes:
  jiovolume:
    name: jiovolume
```

---

# Explanation of Docker Compose File

## version

```yaml
version: "3"
```

Defines Docker Compose file version.

---

## services

```yaml
services:
```

All containers are defined under services.

---

## springapp

```yaml
springapp:
```

Defines Spring Boot application container.

---

## image

```yaml
image: springimage
```

Specifies Docker image name.

---

## ports

```yaml
ports:
  - "8080:8080"
```

## Format

```bash
HOST_PORT:CONTAINER_PORT
```

---

## depends_on

```yaml
depends_on:
  - mongo
```

Starts MongoDB before Spring Boot application.

---

## environment

```yaml
environment:
```

Used to pass environment variables.

### Important Point

```bash
MONGO_DB_HOSTNAME=mongo
```

Here `mongo` is:

* Service Name
* Container Hostname
* Used for container-to-container communication

---

## volumes

```yaml
volumes:
  - jiovolume:/data/db
```

Provides persistent storage for MongoDB data.

Even if the container is deleted, database data remains safe.

---

## networks

```yaml
networks:
  jionetwork:
    driver: bridge
```

Creates custom bridge network.

Both containers communicate inside this network.

---

# How to Save Docker Compose File

Save file as:

```bash
docker-compose.yaml
```

---

# How to Check Docker Compose Version

## Old Syntax

```bash
docker-compose version
```

## New Syntax

```bash
docker compose version
```

---

# How to Install Docker Compose

## Ubuntu / Debian

```bash
sudo apt install docker-compose
```

---

# How to Check Docker Compose File Syntax

```bash
docker-compose config
```

## Checks

* YAML Syntax
* Indentation
* Configuration Validity

---

# How to Run Docker Compose

## Old Syntax

```bash
docker-compose up -d
```

## New Syntax

```bash
docker compose up -d
```

---

# How to Stop Containers

## Old Syntax

```bash
docker-compose down
```

## New Syntax

```bash
docker compose down
```

---

# How to See Containers

## Old Syntax

```bash
docker-compose ps -a
```

## New Syntax

```bash
docker compose ps -a
```

---

# How to See Images

## Old Syntax

```bash
docker-compose images
```

## New Syntax

```bash
docker compose images
```

---

# Useful Docker Compose Commands

| Command                  | Purpose                |
| ------------------------ | ---------------------- |
| `docker compose up -d`   | Start Containers       |
| `docker compose down`    | Stop Containers        |
| `docker compose ps`      | Show Containers        |
| `docker compose logs`    | Show Logs              |
| `docker compose restart` | Restart Services       |
| `docker compose stop`    | Stop Services          |
| `docker compose start`   | Start Stopped Services |
| `docker compose images`  | Show Images            |
| `docker compose config`  | Validate Syntax        |

---

# Important Notes

* Docker Compose creates containers, networks, and volumes automatically.
* Service names act as hostnames inside the Docker network.
* `depends_on` only controls startup order, not application readiness.
* YAML files are indentation sensitive.
* Use spaces instead of tabs in YAML.

---

# Docker Compose Workflow

## Step 1: Create docker-compose.yaml

```bash
vi docker-compose.yaml
```

---

## Step 2: Validate YAML File

```bash
docker compose config
```

---

## Step 3: Start Services

```bash
docker compose up -d
```

---

## Step 4: Check Running Containers

```bash
docker compose ps
```

---

## Step 5: Check Logs

```bash
docker compose logs
```

---

## Step 6: Stop Services

```bash
docker compose down
```

---

# Interview Questions

## What is Docker Compose?

Docker Compose is a tool used to define and manage multi-container Docker applications using a YAML configuration file.

---

## What is the default Docker Compose file name?

```bash
docker-compose.yaml
```

or

```bash
docker-compose.yml
```

---

## What is the use of depends_on?

`depends_on` starts dependent services before the application service.

Example:

```yaml
depends_on:
  - mongo
```

---

## What is the difference between docker-compose and docker compose?

| docker-compose         | docker compose         |
| ---------------------- | ---------------------- |
| Old standalone command | New Docker CLI plugin  |
| Separate installation  | Integrated with Docker |

---

## Can Docker Compose create networks automatically?

Yes.

Docker Compose automatically creates networks for services.

---

## Can Docker Compose create volumes automatically?

Yes.

Docker Compose automatically creates named volumes defined in the YAML file.

---

# Summary

Docker Compose is used to:

* Manage multi-container applications
* Simplify Docker commands
* Automate networking
* Automate volume creation
* Improve application management
* Start and stop applications easily using one YAML file
