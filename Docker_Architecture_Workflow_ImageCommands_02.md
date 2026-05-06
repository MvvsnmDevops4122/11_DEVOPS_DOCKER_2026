#  Docker Architecture Overview

<img width="922" height="484" alt="image" src="https://github.com/user-attachments/assets/9ca751ec-cdd2-4bbd-9ceb-eacdeec4a7f7" />

---

##  What is Docker Architecture?

* Docker Architecture is the system design that defines how Docker functions. 

   * It explains how Docker can create, run, and manage containers efficiently.

   * Docker Architecture Contains Three Main Components:

     1. Docker Client    2. Docker Host   3. Docker Registry
---

##  Docker Client

 🔸 The Docker Client is a command-line interface (CLI) tool.

 🔸 When a user runs commands like docker build, docker pull, or docker run,
    the Docker Client sends an API request to the Docker Daemon inside the Docker Host.

 🔸 The Docker Daemon receives these requests from the Client and executes them,
     managing container operations as needed.


---

## 2️ Docker Host

🔹 The Docker Host is the machine (or environment) where the Docker Daemon and other components like Docker images and containers are managed and run. 

🔹 It is a key part of Docker’s architecture.

###  Docker Daemon (dockerd)

🔸 The Docker Daemon is the main service inside the Docker Host. 

🔸 It waits for instructions from the Docker Client and performs the requested tasks. 

🔸 It can build images from Dockerfiles, run containers based on those images, and manage containers by starting, stopping, or removing them as needed.
      
---

###  Docker Images

🔸 Docker images are templates (read-only snapshots) used to create containers.

🔸 They include everything needed to run an application — such as code, libraries, and system tools.

🔸 Immutable: Once built, they cannot be changed.

🔸 Layered: Built from multiple layers, making them efficient and reusable.

🔸 Version-controlled: You can track changes and roll back to earlier versions easily.

---

###  Docker Containers

🔹  A Docker container is a running instance of a Docker image.

🔹  Containers are lightweight, portable, and run applications in isolated environments.

🔹  Created from Docker images using docker run.

```bash
docker run
```

---

## 3️ Docker Registry

🔸 A registry is a centralized storage system for Docker images.

🔸 Public Registry: Docker Hub (default)

🔸 Private Registries: Nexus, JFrog Artifactory, AWS ECR

### Types:

* Public → Docker Hub
* Private → Nexus, JFrog, AWS ECR

---

#  Docker Workflow

**Dockerfile → Docker Image → Docker Container**

<img width="979" height="478" alt="image" src="https://github.com/user-attachments/assets/d8153e8b-1a18-4499-acc4-2397bf024507" />

---

##  1. Dockerfile

  🔹 A Dockerfile is a blueprint that contains a set of instructions to build a Docker image. 

  🔹 It decribes all the steps, base image, files, dependencies, configuration, environment settings, and commands needed to package and 

      run your application inside a Docker container.

---

##  2. Docker Images

  🔸 Docker images are templates (read-only snapshots) used to create containers.

  🔸 They include everything needed to run an application — such as code, libraries, and system tools.

  🔸 Immutable: Once built, they cannot be changed.

  🔸 Layered: Built from multiple layers, making them efficient and reusable.

  🔸 Version-controlled: You can track changes and roll back to earlier versions easily.

   Create the docker image : 
   ```bash
    docker build
   ```

---

##  3. Docker Containers

  🔹  A Docker container is a running instance of a Docker image.
   
  🔹  Containers are lightweight, portable, and run applications in isolated environments.

  Create the docker container : docker run/docker create

```bash
docker run
docker create
```

---

#  Docker Objects

# Summary of Docker Objects

Containers:      Running instances of images.

Images:          Read-only templates used to create containers.

Volumes:         Persistent storage for container data.

Networks:        Facilitate communication between containers and external systems.

Registries:      Storage and distribution points for images.

Dockerfile:      Defines how to build a Docker image.

Docker Compose:  Manages multi-container Docker applications.

---

#  Docker Build Context

 * In Docker, the build context is the collection of files and directories available to the Docker engine

   when building a Docker image. The build context includes the Dockerfile and any files needed for the image build process.

---

#  Practical Session

## Step 1: Verify Docker Installation

```bash
docker --version
```

---

## Step 2: Clone a Sample Web Application

```bash
git clone https://github.com/MvvsnmDevops4122/05_DEVOPS_MAVEN-WEBAPPLICATION-PROJECT_2026.git
```

---

## Step 3: Install Maven and Java

```bash
sudo apt install maven -y  # Installs Maven and JRE
java --version             # Verify Java installation
mvn -version               # Verify Maven installation
```

---

## Step 4: Build the Application Artifact

* Ensure your pom.xml contains a valid Maven build configuration:
```bash
    <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>3.3.2</version> <!-- Use a stable version -->
    </plugin>
```

```bash
mvn clean package
```

---

## Step 5: Create Dockerfile
```
   vi Dockerfile
```
Add the following content to the Dockerfile:

```dockerfile
FROM tomcat:9.0.100
COPY target/maven-web-application.war /usr/local/tomcat/webapps/maven-web-application.war
```

---

## Step 6: Build Image

```bash
docker build -t satyamolleti4599/maven-web-app:1.0.0 .
docker history satyamolleti4599/maven-web-app:1.0.0
```

---

## Step 7: Verify Images

```bash
docker images
docker image ls
```

---

## Step 8: Push Image to Docker Hub

Before pushing, authenticate to Docker Hub:
```bash
docker login -u satyamolleti4599
docker push satyamolleti4599/maven-web-app:1.0.0
```

---

## Step 9: Run a Container from the Image

```bash
docker run -d -p 8080:8080 --name javawebapp satyamolleti4599/maven-web-app:1.0.0
```
-d == detching mode, -p == port forwarding , 8080 = hostport, 8080= container port, javawebapp=container name 

---

## Step 10: Verify Running Containers

```bash
docker ps -a  # Lists all containers
```

---

##  Logout

```bash
docker logout
```

---

# Docker Image Commands 

##  Checking Docker Containers

```bash

    docker ps  # Lists running containers

    docker ps -a # List running and stopped containers

    docker container ls  # Alternative to 'docker ps'  
```

---

##  Running Containers (create)

```bash
docker run -d -p 80:80 --name nginxcontainer nginx:latest
docker run -d -p 8080:8080 --name tomcatcontainer tomcat:latest
```

---

##  Removing and Restarting a Container

```bash
docker rm -f container_id/container_name
docker run -d -p 8080:8080 --name tomcat tomcat:6.0.43-jre8
```

---

## Docker info & docker version

```bash
docker info
docker --version or docker version
```

---

## How to Build a Docker Image

```bash
docker build -t <image_name>:<tag> <build_context> (New image name)
```
   Examples:

   docker build -t satyamolleti4599/maven-web-app:1.0.0 .

   docker build -t nexus.jio.com/java-web-app:1 .
   
---

## How to List Docker Images  

```bash
docker images
docker image ls
```

---

## How to Login to Docker Hub

```bash
docker login -u <username> -p <password>
```
Ex: docker login -u satyamolleti4599

---

##  How to Push a Docker Image

```bash
docker push satyamolleti4599/maven-web-app:1.0.0
```

---

##  How to Pull a Docker Image

```bash
docker pull satyamolleti4599/maven-web-app:1.0.0
```

---

## How to Inspect an Image

```bash
docker image inspect <image_id or image_name>           #More info about image

docker inspect --format='{{.Id}}' e52c7cf56da7          #Get just the image ID 

     (OR) docker inspect -f '{{.Id}}' e52c7cf56da7

docker inspect --format='{{.RepoTags}}' e52c7cf56da7    #Get image tags (RepoTags)

docker inspect --format='{{.Created}}' e52c7cf56da7     # Get created time

docker inspect --format='{{json .Config.ExposedPorts}}' e52c7cf56da7  #Get exposed ports

docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' e52c7cf56da7  # Get environment variables
```

---

## To See the Layers of an Image

```bash
docker history <image_id or image_name>

```
Ex: docker history satyamolleti4599/maven-web-app:1.0.0

---

## 🗑️ Delete Images

```bash
Single image:   docker rmi <image_id or image_name>
Multiple images: docker rmi <image1> <image2>
All images: docker rmi -f $(docker images -aq)
```
docker images -aq---> we can get all imageIDs

---

## Tagging Docker Images(Rename)

```bash
docker tag <source_image>:<tag> <new_image>:<new_tag>
```
EX: docker tag ubuntu:latest satyamolleti4599/kkubuntu:latest

---

# How to move image from one server to another server?

## Approach 1 (Docker Hub)

1. push image from server1 to docker hub
2. Pull and run image from docker hub to server2.

---

## Approach 2 (Tar File)

### Step 1: create tar file for that image

```bash
 docker save -o <filename>.tar <imageId>
 docker save -o webapp.tar satyamolleti4599/maven-web-app:1.0.0
```

### Step 2: take tar file to local by scp command

```bash
scp -i ~/Downloads/KKDevops.pem ubuntu@13.233.2.78:/home/ubuntu/Maven-Web_Application-/localhost.2025-07-27.log ~/Downloads/
```

### Step 3: Move tar file from local to server2

```bash
scp -i ~/Downloads/KKDevops.pem ~/Downloads/webapp.tar ubuntu@13.232.78.253:/home/ubuntu/
```

### Step 4: untar the file inside server2

```bash
docker load -i webapp.tar
```

---

#  Create a Docker Account

## 1. Go to the Docker Website

* Open your browser and go to:
   https://hub.docker.com

## 2. Click “Sign Up”

* You’ll find the **Sign Up** button in the **upper-right corner**

## 3. Fill Out the Registration Form

* **Username:** Choose a unique Docker ID
* **Email:** Use a valid email address
* **Password:** Create a strong password

## 4. Agree to Terms and Conditions

* Check the box to agree to Docker’s **terms of service and privacy policy**

## 5. Click “Sign Up”

* After completing the form, click the **Sign Up** button

## 6. Verify Your Email

* Check your inbox for a confirmation email from Docker
* Click the link in the email to verify your account
---
