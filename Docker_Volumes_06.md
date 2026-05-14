# DOCKER VOLUMES

## Docker Volumes

* Containers are temporary. When they stop or restart, their data is lost.
* Volumes store data outside the container, keeping it safe even if the container is deleted or crashed.
* Volumes are used to save data created by containers.
* They help prevent data loss.
* Volumes make containers stateful, meaning data stays even after restarts.

## Types of Volumes in Docker

### 1) Bind Mounts (Host Mount)

By using a bind mount, we can store data in the server's file system. Even if our container crashes, we can still recover the data.

### 2) Volumes (External Volumes)

* Local Volumes
* Network Volumes

  * NFS
  * EBS
  * S3

---

# Before Docker Volumes

* Earlier, data was stored inside the container’s file system.
* If the container crashed or was removed, all data would be lost.
* To solve this problem, Docker volumes were introduced.
* Volumes store data outside the container, keeping it safe even if the container is deleted.

---

# Revised LAB: Spring Boot + MongoDB (Before Docker Volumes)

## Step 1: Fork the Project

Fork the repo into your GitHub account:

```bash
https://github.com/MvvsnmDevops4122/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026.git
```

## Step 2: Clone the Project

```bash
git clone https://github.com/MvvsnmDevops4122/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026.git

cd 13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026
```

## Step 3: Check the Dockerfile

```bash
cat Dockerfile
```

### Key Content

```Dockerfile
# Stage 1: Build the application
FROM maven:3.8.5-openjdk-8-slim AS build

# Set the working directory inside the container
WORKDIR /app

# Copy the pom.xml file
COPY pom.xml .

# Download dependencies
RUN mvn dependency:go-offline -B

# Copy source code
COPY src ./src

# Build the application
RUN mvn clean package -DskipTests

# Stage 2: Run the application
FROM eclipse-temurin:8-jdk-alpine

# Set environment variable
ENV PROJECT_HOME=/opt/app

# Set working directory
WORKDIR $PROJECT_HOME

# Copy JAR from build stage
COPY --from=build /app/target/spring-boot-mongo-1.0.jar spring-boot-mongo.jar

# Expose application port
EXPOSE 8080

# Run application
CMD ["java", "-jar", "spring-boot-mongo.jar"]
```

## Step 4: Build the Docker Image

```bash
docker build -t springimage .

docker images
```

## Step 5: Create a Custom Docker Bridge Network

```bash
docker network create jionetwork

docker network ls
```

## Step 6: Create Spring Boot Application Container

```bash
docker run -d -p 8080:8080 --name springappcon --network jionetwork \
-e MONGO_DB_HOSTNAME=mongo \
-e MONGO_DB_USERNAME=devdb \
-e MONGO_DB_PASSWORD=dev@123 \
springimage
```

### Notes

* `-e` = Environment Variables
* In Spring Boot, Tomcat is embedded by default.
* Default MongoDB Port: `27017`
* We can get the above details from:

```bash
cat src/main/resources/application.yml
```

* `MONGO_DB_HOSTNAME=mongo` value should be the same as the MongoDB container name.
* Database username and password values are custom values.

## Step 7: Access the Application

Open browser:

```bash
http://<your-ec2-ip>:8080
```

Example:

```bash
http://13.126.86.86:8080
```

* Data will not be saved yet because MongoDB is not running.

## Step 8: Create MongoDB Container

```bash
docker pull mongo:6.0
```

```bash
docker run -d --name mongo --network jionetwork \
-e MONGO_INITDB_ROOT_USERNAME=devdb \
-e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
mongo:6.0
```

### Notes

* Now you can insert data.
* If the Mongo image is not available locally, Docker pulls it from Docker Hub.
* `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD` are official MongoDB image environment variables.

## Step 9: Check MongoDB for Data

### Enter into MongoDB Container

```bash
docker exec -it mongo bash
```

### Open Mongo Shell

```bash
mongosh --host localhost:27017 -u devdb -p dev@123
```

### Clear Screen in Mongo Shell

```bash
cls
```

### MongoDB Commands

```bash
show databases;
```

```bash
use users;
```

```bash
db.users.find();
```

### Exit Commands

```bash
exit
```

```bash
exit
```

## Step 10: See the Default Location of MongoDB

```bash
docker exec mongo ls /data/db
```

## Step 11: Delete App Container — Check If Data Still Exists

### Delete Only Spring App Container

```bash
docker rm -f springapp
```

### Recreate Spring App Container

```bash
docker run -d -p 8080:8080 --name springapp --network jionetwork \
-e MONGO_DB_HOSTNAME=mongo \
-e MONGO_DB_USERNAME=devdb \
-e MONGO_DB_PASSWORD=dev@123 \
springimage
```

### Verification

Open:

```bash
http://localhost:8080
```

MongoDB still holds the data because the MongoDB container was not deleted.

## Step 12: Delete MongoDB Container — Check If Data Is Lost

### Remove MongoDB Container

```bash
docker rm -f mongo
```

### Recreate MongoDB Container

```bash
docker run -d --name mongo --network jionetwork \
-e MONGO_INITDB_ROOT_USERNAME=devdb \
-e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
mongo:6.0
```

### Verification

* Check old user data using Spring Boot app or Mongo shell.
* Data will be lost.

### Why?

MongoDB data was stored inside the container at:

```bash
/data/db
```

Since no volume was used, deleting the container removed the data.

---

# Why We Need Volumes

## Step 13: Use Docker Volumes for Persistence

### Create Named Volume

```bash
docker volume create mongovolume
```

### List Volumes

```bash
docker volume ls
```

### Inspect Volume

```bash
docker volume inspect mongovolume
```

### Sample Output

```json
[
  {
    "Name": "mongovolume",
    "Driver": "local",
    "Mountpoint": "/var/lib/docker/volumes/mongovolume/_data"
  }
]
```

### Notes

* Mountpoint is the actual location where Docker stores data on the host machine.

### Check Volume Data on Host

```bash
sudo ls /var/lib/docker/volumes/mongovolume/_data
```

## Create MongoDB Container with Volume

```bash
docker run -d --name mongo --network jionetwork \
-e MONGO_INITDB_ROOT_USERNAME=devdb \
-e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
-v mongovolume:/data/db \
mongo
```

### Notes

* `/data/db` inside the container is mapped to a persistent volume.

## Test the Setup

1. Connect with Spring app.
2. Insert some user data.
3. Remove MongoDB container.

```bash
docker rm -f mongo
```

4. Recreate MongoDB with same volume.

```bash
docker run -d --name mongo --network jionetwork \
-e MONGO_INITDB_ROOT_USERNAME=devdb \
-e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
-v mongovolume:/data/db \
mongo
```

### Result

Data still exists because volume stores data separately from the container.

---

# Bind Mount

## What is a Bind Mount?

* A bind mount maps a file or directory from the host system to the container file system.
* By using a bind mount, we can store data in the server file system.
* Even if the container crashes, data can still be recovered.

### Important Note

To map the host file system with the container file system, use the `-v` option.

### Syntax

```bash
docker run -v /host/path:/container/path
```

## Example

```bash
docker run -d --name mongo --network jionetwork \
-e MONGO_INITDB_ROOT_USERNAME=devdb \
-e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
-v /home/ubuntu/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026/backup:/data/db \
mongo:6.0
```

---

# Steps to Use Bind Mount for MongoDB

## Step 1: Create Backup Directory on Host

```bash
mkdir -p /home/ubuntu/Maven-Web_Application-/B5-projects/spring-boot-mongo-docker-kkfunda/backup
```

## Step 2: Navigate to Directory

```bash
cd /home/ubuntu/Maven-Web_Application-/B5-projects/spring-boot-mongo-docker-kkfunda/backup
```

## Step 3: Run MongoDB with Bind Mount

```bash
docker run -d --name mongo --network jionetwork \
-e MONGO_INITDB_ROOT_USERNAME=devdb \
-e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
-v /home/ubuntu/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026/backup:/data/db \
mongo:6.0
```

## Step 4: Insert Data into MongoDB

Insert data from Spring Boot application.

## Step 5: Remove MongoDB Container

```bash
docker rm -f mongo
```

## Step 6: Recreate MongoDB Container with Same Bind Mount

```bash
docker run -d --name mongo --network jionetwork \
-e MONGO_INITDB_ROOT_USERNAME=devdb \
-e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
-v /home/ubuntu/Maven-Web_Application-/B5-projects/spring-boot-mongo-docker-kkfunda/backup:/data/db \
mongo:6.0
```

## Step 7: Verify Data

MongoDB data is restored because it is persisted on the host machine.

---

# Assignment: Run Jenkins in Docker with Volumes and Create a Job

## Step 1: Pull Jenkins Docker Image

```bash
docker pull jenkins/jenkins:lts
```

### Notes

* `lts` = Long Term Support version.

## Step 2: Create Docker Volume for Jenkins

```bash
docker volume create jenkins_data
```

## Step 3: Run Jenkins Container with Named Volume

```bash
docker run -d -p 8080:8080 --name jenkins \
-v jenkins_data:/var/jenkins_home \
jenkins/jenkins:lts
```

## Step 4: Get Jenkins Initial Password

```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Open Browser:

```bash
http://13.126.86.86:8080
```

## Step 5: Setup Jenkins UI

* Install Suggested Plugins
* Create Admin User
* Skip optional plugins if required

## Step 6: Create Jenkins Job

### Steps

1. Create Freestyle Project
2. Under Build → Execute Shell:

```bash
echo "Testing volume persistence"
```

3. Save and Build Now

## Step 7: Verify Volume Persistence

### Remove Jenkins Container

```bash
docker rm -f jenkins
```

### Recreate Jenkins Container

```bash
docker run -d -p 8080:8080 --name jenkins \
-v jenkins_data:/var/jenkins_home \
jenkins/jenkins:lts
```

### Result

* Jenkins jobs still exist.
* Plugins and configurations remain available.

---

# Interview Questions

## Why do we use Docker Volumes instead of Bind Mounts?

### Answer

We take backups for each container in different folders. Since there is no central place for backup, Docker volumes provide centralized and better-managed storage.

---

## How can you migrate Docker from one server to another?

### Answer

You can migrate Docker containers, images, and volumes by archiving Docker data and transferring it.

### Source Server

```bash
sudo tar -czvf docker-backup.tar.gz /var/lib/docker
```

### Transfer Backup

```bash
scp docker-backup.tar.gz user@target-server:/tmp
```

### Target Server

```bash
sudo systemctl stop docker
```

```bash
sudo tar -xzvf /tmp/docker-backup.tar.gz -C /
```

```bash
sudo systemctl start docker
```

### Note

Docker versions should be compatible on both servers.

---

# Docker Volume Management

## List Docker Volumes

```bash
docker volume ls
```

## Remove Unused Volumes

```bash
docker volume prune
```

### Notes

This removes anonymous local volumes not used by any container.

## Remove Specific Volume

```bash
docker volume rm <volume_name>
```

## Create Named Volume

```bash
docker volume create -d local jiovolume
```

```bash
docker volume create Airtelvolume
```

```bash
docker volume ls
```

## Inspect Docker Volume

```bash
docker volume inspect jiovolume
```

### Sample Output

```json
[
  {
    "CreatedAt": "2026-05-14T10:13:09Z",
    "Driver": "local",
    "Mountpoint": "/var/lib/docker/volumes/jiovolume/_data",
    "Name": "jiovolume",
    "Options": null,
    "Scope": "local"
  }
]
```

## Docker Volume Location

```bash
sudo ls -lth /var/lib/docker/volumes/
```

---

# Running Containers with Volumes

## Run MongoDB with Named Volume

```bash
docker run -d --name mongo \
-v jiovolume:/data/db \
--network jionetwork \
-e MONGO_INITDB_ROOT_USERNAME=devdb \
-e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
mongo:6.0
```

## Run Spring Boot Application

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

# Data Persistence Verification

## Real-Time Scenario

### Step 1

Run MongoDB container with named volume.

### Step 2

Insert sample data using Spring Boot application.

### Step 3

Remove MongoDB container.

```bash
docker rm -f mongo
```

### Step 4

Recreate MongoDB container using same volume.

```bash
docker run -d --name mongo \
-v jiovolume:/data/db \
--network jionetwork \
-e MONGO_INITDB_ROOT_USERNAME=devdb \
-e MONGO_INITDB_ROOT_PASSWORD=dev@123 \
mongo:6.0
```

### Result

Data is still present because volume stores data separately from the container.

---

# Interview Question

## Can I map a volume or port to a running container?

### Answer

No.

You cannot add port mappings or volume mappings to a running container after it has started.
