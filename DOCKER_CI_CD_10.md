# Building CI/CD Pipeline: Jenkins & SonarQube Docker Image on CI Server, Remote Docker Deployment via CD Server

---

# 1. Launch Ubuntu Server (t2.large)

Used as:

* Jenkins CI Server
* SonarQube Server

## Instance Type

```text
t2.large or t3.large
```

```text
2 vCPU, 8 GB RAM
```

---

# 2. Launching a Jenkins CI Server

Connect to your console and install Jenkins.

Run this script in root user:

```bash
sudo su -
```

Create script:

```bash
vi jenkins.sh
```

---

# Jenkins Installation Script

```bash
#!/bin/bash

#################################################
# Jenkins Installation Script - Ubuntu
# Author : Satya
# Purpose: Install Jenkins with Java 21
#################################################

echo "=========================================="
echo " Updating Package Repository"
echo "=========================================="

sudo apt update -y

echo
echo "=========================================="
echo " Installing Java 21"
echo "=========================================="

sudo apt install -y fontconfig openjdk-21-jre

echo
echo "=========================================="
echo " Java Version"
echo "=========================================="

java -version

echo
echo "=========================================="
echo " Adding Jenkins Repository Key"
echo "=========================================="

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo
echo "=========================================="
echo " Adding Jenkins Repository"
echo "=========================================="

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/" | \
sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

echo
echo "=========================================="
echo " Updating Repository Again"
echo "=========================================="

sudo apt update -y

echo
echo "=========================================="
echo " Installing Jenkins"
echo "=========================================="

sudo apt install -y jenkins

echo
echo "=========================================="
echo " Enabling Jenkins Service"
echo "=========================================="

sudo systemctl enable jenkins

echo
echo "=========================================="
echo " Starting Jenkins Service"
echo "=========================================="

sudo systemctl start jenkins

echo
echo "=========================================="
echo " Jenkins Service Status"
echo "=========================================="

sudo systemctl status jenkins --no-pager

echo
echo "=========================================="
echo " Jenkins Installation Completed"
echo "=========================================="

echo "Access Jenkins using:"
echo "http://<SERVER-IP>:8080"

echo
echo "To Get Jenkins Initial Password:"
echo

sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

# 3. Installing Docker in Jenkins and Configuring Permissions

# 1. Install Docker

## Update Package Repository

```bash
sudo apt-get update
```

### Purpose

* Updates package repository information.

---

## Install Docker

```bash
sudo apt-get install docker.io -y
```

### Purpose

* Installs Docker.
* `-y` automatically accepts installation prompts.

---

# 2. Add Current User to Docker Group

## Add User

```bash
sudo usermod -aG docker $USER
```

### Purpose

* Adds current user (`ubuntu` or `ec2-user`) to Docker group.
* Allows running Docker commands without sudo.

---

## Apply Docker Group Permissions Immediately

```bash
newgrp docker
```

### Purpose

* Applies Docker group permissions immediately.

---

## Give Docker Socket Permissions

```bash
sudo chmod 777 /var/run/docker.sock
```

### Purpose

* Gives full permissions to Docker socket file.
* Fixes Docker permission denied issues.
* Docker client communicates with Docker daemon using this socket file.

### Important Note

* Not recommended in production environments due to security risks.

---

# 3. Verify Docker Installation

```bash
docker info
```

### Purpose

* Displays Docker system information.
* Confirms Docker is working properly.

---

# 4. Give Docker Permissions to Jenkins User

```bash
sudo usermod -aG docker jenkins
```

### Purpose

* Adds Jenkins user to Docker group.
* Allows Jenkins pipelines to execute Docker commands.

---

# Check Jenkins User Groups

```bash
groups jenkins
```

### Purpose

* Displays groups assigned to Jenkins user.

---

# Before Adding Docker Group

```text
jenkins : jenkins
```

### Meaning

* Jenkins belongs only to its own group.
* Cannot run Docker commands yet.

---

# 5. Verify Docker Group Members

```bash
getent group docker
```

## Example Output

```text
docker:x:998:ubuntu,jenkins
```

### Meaning

* Confirms Jenkins user is added to Docker group successfully.

---

# 6. Restart Jenkins Service

```bash
sudo systemctl restart jenkins
```

### Purpose

* Restarts Jenkins service.
* Applies new Docker group permissions.

---

# 7. Final Verification

Run inside Jenkins pipeline or terminal:

```bash
docker ps
```

or

```bash
docker version
```

### Result

* If output is displayed successfully, Jenkins can now use Docker properly.

---

# Complete Flow

```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
# 4. Create a Sonarqube Container

## Run SonarQube Container

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

### Important Note

* Enable port `9000` in the AWS Security Group.

---

# Install Required Plugins in Jenkins

## Docker Plugins

* Docker
* Docker Pipeline
* Docker Build Step

## SonarQube Plugins

* SonarQube Scanner
* Quality Gates Plugin

---

# 5. Creating a Jenkins Job (Pipeline)

A Jenkins Pipeline Job is used to automate the CI/CD process.

---

# Step 1: Create a New Jenkins Pipeline Job

* Go to Jenkins Dashboard.
* Click:

```text
New Item
```

* Give your job a name.

Example:

```text
jenkin-docker
```

* Select:

```text
Pipeline
```

* Click:

```text
OK
```

---

# Step 2: Configure Maven

Jenkins needs Maven to build Java applications.

Jenkins downloads Maven automatically.

## Navigation

```text
Manage Jenkins → Global Tool Configuration
```

## Maven Configuration Steps

* Under Maven click:

```text
Add Maven
```

* Name:

```text
mvn_3.9.15
```

* Select:

```text
Install automatically
```

* Choose Maven Version:

```text
3.9.15
```

* Click Save.

---

# Step 3: Write the Jenkins Pipeline Script

## Goal

* Clone code from GitHub
* Build project using Maven
* Verify Jenkins pipeline is working correctly

---

# Jenkins Pipeline Script

```groovy
pipeline {
    agent any

    tools {
        maven 'mvn_3.9.15' // This should match the Maven name in Jenkins Global Tool Configuration
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/MvvsnmDevops4122/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

    }
}
```

---

# Explanation

## 1. pipeline

Defines Jenkins Declarative Pipeline.

---

## 2. agent any

Pipeline can run on any available Jenkins agent/node.

---

## 3. Tools Section

```groovy
tools {
    maven 'mvn_3.9.15'
}
```

### Purpose

* Uses Maven configured in Jenkins.
* `mvn_3.9.15` must match Maven name in:

```text
Manage Jenkins → Global Tool Configuration
```

---

## 4. Stage: Checkout

Used to download source code from GitHub.

### No Credentials Required

* Repository is public.
* Jenkins can access directly.

---

## 5. Stage: Build

Compiles and packages application.

---

# 6. Integrating SonarQube for Static Code Analysis

# Step 1: Generate SonarQube Token

## Open SonarQube Dashboard

```text
http://<SERVER-IP>:9000
```

---

## Login Credentials

```text
Username: admin
Password: admin
```

---

## Generate Token

### Navigation

```text
Profile → My Account → Security
```

### Token Details

Name:

```text
jenkins-token
```

### Steps

1. Click:

```text
Generate
```

2. Copy token immediately.

### Important Note

* You cannot see the token again later.

---

# Step 2: Configure SonarQube Server in Jenkins

## Navigation

```text
Manage Jenkins → Configure System
```

---

## SonarQube Server Configuration

Scroll to:

```text
SonarQube Servers
```

Click:

```text
Add SonarQube
```

### Fill Details

| Field      | Value                                                                |
| ---------- | -------------------------------------------------------------------- |
| Name       | sonarQube                                                            |
| Server URL | [http://your-sonarqube-server-url](http://your-sonarqube-server-url) |

---

# Add SonarQube Token

Under:

```text
Authentication Token
```

Click:

```text
Add → Jenkins
```

## Fill Details

| Field       | Value                 |
| ----------- | --------------------- |
| Kind        | Secret text           |
| Secret      | Paste SonarQube Token |
| Description | sonartoken            |

Click:

```text
Save
```

Now select:

```text
sonartoken
```

from dropdown.

Save Jenkins configuration.

---

# Important Note

If token is not added yet:

```text
Add → Jenkins → Jenkins Credentials Provider
```

---

# Step 3: Configure Sonar Scanner

## Navigation

```text
Manage Jenkins → Global Tool Configuration
```

Scroll to:

```text
SonarQube Scanner
```

Click:

```text
Add SonarQube Scanner
```

## Fill Details

| Field                 | Value        |
| --------------------- | ------------ |
| Name                  | SonarScanner |
| Install Automatically | Enabled      |

Click:

```text
Save
```

---

# Result

Now:

* Jenkins connected with SonarQube
* Ready for static code analysis
* Ready to use SonarQube in Jenkins pipeline

---

# Step 4: Run Jenkins Build with SonarQube Integration

Update Jenkins pipeline with SonarQube stage.

---

# Jenkins Pipeline Script with SonarQube

```groovy
pipeline {

    agent any

    tools {
        maven 'mvn_3.9.15'
    }

    stages {

        stage('Checkout') {
            steps {

                git branch: 'master',
                url: 'https://github.com/MvvsnmDevops4122/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026.git'

            }
        }

        stage('Build') {
            steps {

                sh 'mvn clean package'

            }
        }

        stage('SonarQube') {
            steps {

                withSonarQubeEnv('sonarQube') {

                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=spring-boot-mongo \
                    -Dsonar.projectName="Spring Boot Mongo Project"
                    '''

                }
            }
        }
    }
}
```

---

# Explanation

## withSonarQubeEnv('sonarQube')

### Purpose

* Connects Jenkins with SonarQube server.
* `sonarQube` must match:

```text
Manage Jenkins → Configure System → SonarQube Servers
```

### Jenkins Automatically Uses

* SonarQube URL
* Authentication Token

No need to manually pass token.

---

# Generate SonarQube Snippet

## Steps

1. Open Jenkins Dashboard
2. Open Pipeline Job
3. Click:

```text
Pipeline Syntax
```

4. Select:

```text
withSonarQubeEnv: Prepare SonarQube Scanner environment
```

5. Enter SonarQube server name:

```text
sonarQube
```

This name must match:

```text
Manage Jenkins → Configure System → SonarQube Servers
```

6. Click:

```text
Generate Pipeline Script
```

Generated Snippet:

```groovy
withSonarQubeEnv('sonarQube')
{

}
```

---

# SonarQube Command

```bash
mvn sonar:sonar
```

### Purpose

Runs static code analysis.

---

# Project Key

```bash
-Dsonar.projectKey=spring-boot-mongo-satya
```

### Purpose

* Unique project identifier in SonarQube.
* You can provide any unique name.

---

# Project Name

```bash
-Dsonar.projectName="Spring Boot Mongo satya Project"
```

### Purpose

Display name in SonarQube Dashboard.

---

# 7. Configuring Docker Authentication in Jenkins

If you want Jenkins to push Docker images to Docker Hub, you need:

* Docker Hub Username
* Docker Hub Token

---

# Step 1: Generate Docker Hub Token

## Login to Docker Hub

---

## Navigation

```text
Profile → Account Settings → Personal Access Tokens
```

---

## Generate Token

Click:

```text
Generate New Token
```

### Token Permissions

* Read
* Write
* Delete

Click:

```text
Generate
```

---

# Step 2: Add Docker Hub Credentials to Jenkins

## Navigation

```text
Jenkins Dashboard → Manage Jenkins → Credentials
```

---

# Add Credentials

1. Open:

```text
Global credentials
```

2. Click:

```text
Add Credentials
```

---

# Fill Credential Details

| Field       | Value                  |
| ----------- | ---------------------- |
| Kind        | Username with password |
| Username    | satyamolleti4599       |
| Password    | Docker Hub Token       |
| ID          | docker                 |
| Description | docker-cred            |

Click:

```text
OK
```

to save the credentials.

---

# 8. Build & Push Docker Image to Docker Hub using Jenkins

---

# Prerequisites

1. Jenkins installed

2. Docker installed

3. Jenkins user added to Docker group

```bash
sudo usermod -aG docker jenkins
```

4. Docker Hub credentials added in Jenkins

5. GitHub repository contains Dockerfile

---

# Step 1: Generate Docker Snippet

Open Jenkins Job.

Go to:

```text
Pipeline Syntax
```

Select:

```text
Snippet
```

From dropdown select:

```text
withDockerRegistry: Sets up Docker registry endpoint
```

## Configuration

| Field        | Value       |
| ------------ | ----------- |
| Registry URL | Leave Empty |
| Credentials  | docker      |

---

# Generate Script

Click:

```text
Generate Pipeline Script
```

Generated Snippet:

```groovy
withDockerRegistry(credentialsId: 'docker') {
// some block
}
```

---

# Step 2: Jenkins Pipeline Script

```groovy
pipeline {
    agent any

    tools {
        maven 'mvn_3.9.15' // This should match the Maven name in Jenkins Global Tool Configuration
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/MvvsnmDevops4122/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
       
        stage('SonarQube') {
            steps {

                withSonarQubeEnv('sonarQube') {

                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=spring-boot-mongo \
                    -Dsonar.projectName="Spring Boot Mongo Project"
                    '''

                }
            }
        }

        stage('Build Docker Image') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh 'docker build -t satyamolleti4599/mongospring:latest .'

                    }
                }
            }
        }
    }
}
```

---

# Explanation

# 1. Docker Build

```bash
docker build -t satyamolleti4599/mongospring:latest .
```

### Purpose

Builds Docker image from Dockerfile.

---

# 2. -t Option

### Purpose

Used to tag Docker image.

Image Name:

```text
satyamolleti4599/mongospring:latest
```

---

# 3. Docker Push

```bash
docker push satyamolleti4599/mongospring:latest
```

### Purpose

Pushes Docker image to Docker Hub repository.

---

# 4. withDockerRegistry

```groovy
withDockerRegistry(credentialsId: 'docker')
```

### Purpose

* Uses Docker Hub credentials securely.
* `docker` must match Jenkins credential ID.

---

# Step 2: Pipeline Script for Building and Pushing Docker Image

```groovy
pipeline {

    agent any

    tools {
        maven 'mvn_3.9.15'
    }

    stages {

        stage('Checkout') {
            steps {

                git branch: 'master',
                url: 'https://github.com/MvvsnmDevops4122/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026.git'

            }
        }

        stage('Build') {
            steps {

                sh 'mvn clean package'

            }
        }

        stage('SonarQube') {
            steps {

                withSonarQubeEnv('sonarQube') {

                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=spring-boot-mongo \
                    -Dsonar.projectName="Spring Boot Mongo Project"
                    '''

                }
            }
        }

        stage('Build Docker Image') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh 'docker build -t satyamolleti4599/mongospring:latest .'

                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh 'docker push satyamolleti4599/mongospring:latest'

                    }
                }
            }
        }
    }
}
```

---

# Step 3: Run the Jenkins Job

Save the pipeline script.

## Steps

1. Click:

```text
Build Now
```

2. Login to Docker Hub.

3. Check Repository:

```text
satyamolleti4599/mongospring
```

### Verification

You should see:

```text
latest
```

tag available successfully.

---

# 9. Configure CD Server for Deployment

---

# 1. Launching the CD Server (t3.micro)

### Purpose

* Create separate EC2 server for deployments.
* Keeps CI and CD environments separate.

---

# 2. Install Docker on CD Server

```bash
sudo apt-get update
```

```bash
sudo apt-get install docker.io -y
```

```bash
sudo usermod -aG docker $USER
```

```bash
newgrp docker
```

```bash
sudo chmod 777 /var/run/docker.sock
```

### Purpose

* Installs Docker
* Adds user to Docker group
* Enables Docker permissions

---

# 10. Setup Jenkins Deployment via SSH

---

# Step 1: Install SSH Agent Plugin

## Navigation

```text
Manage Jenkins → Plugins
```

Install:

```text
SSH Agent Plugin
```

### Purpose

Helps Jenkins connect to remote server using SSH key.

---

# Step 2: Use the .pem File from CD Server

To connect Jenkins to CD server, you need the server SSH private key (`.pem` file).

---

# View the PEM File

Run this command on local machine or server where `.pem` file exists:

```bash
cat /path/to/your-key.pem
```

Example:

```bash
cat Nissi.pem
```

---

# Copy Full Key

Copy everything starting from:

```text
-----BEGIN RSA PRIVATE KEY-----
```

up to:

```text
-----END RSA PRIVATE KEY-----
```

---

# Step 3: Add SSH Key to Jenkins Credentials

## Navigation

```text
Dashboard → Manage Jenkins → Credentials
```

---

# Credential Details

| Field       | Value                          |
| ----------- | ------------------------------ |
| Kind        | SSH Username with private key  |
| ID          | cd-ssh                         |
| Username    | ubuntu                         |
| Private Key | Enter directly → Paste PEM key |

---

# Step 4: SSH Deployment via Jenkins Pipeline

```groovy
pipeline {

    agent any

    tools {
        maven 'mvn_3.9.15'
    }

    stages {

        stage('Checkout') {
            steps {

                git branch: 'master',
                url: 'https://github.com/MvvsnmDevops4122/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026.git'

            }
        }

        stage('Build') {
            steps {

                sh 'mvn clean package'

            }
        }

        stage('SonarQube') {
            steps {

                withSonarQubeEnv('sonarQube') {

                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=spring-boot-mongo \
                    -Dsonar.projectName="Spring Boot Mongo Project"
                    '''

                }
            }
        }

        stage('Build Docker Image') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh 'docker build -t satyamolleti4599/mongospring:latest .'

                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh 'docker push satyamolleti4599/mongospring:latest'

                    }
                }
            }
        }

        stage('Deploy to CD Server') {
            steps {

                sshagent(['cd-ssh']) {

                    sh '''
ssh -o StrictHostKeyChecking=no ubuntu@34.227.48.151 << EOF

docker pull satyamolleti4599/mongospring:latest

docker stop myspringcontainer || true

docker rm myspringcontainer || true

docker run -d \
--name myspringcontainer \
-p 8080:8080 \
--restart unless-stopped \
satyamolleti4599/mongospring:latest

EOF
                    '''

                }
            }
        }
    }
}
```

---

# Explanation

# sshagent(['cd-ssh'])

### Purpose

Uses Jenkins SSH credentials.

`cd-ssh` is Jenkins credential ID containing `.pem` private key.

---

# SSH Command

```bash
ssh -o StrictHostKeyChecking=no ubuntu@13.48.128.52
```

### Purpose

Connects to remote CD server.

| Part          | Meaning                |
| ------------- | ---------------------- |
| ubuntu        | Remote server username |
| 34.227.48.151 | Remote server IP       |

---

# Docker Pull

```bash
docker pull satyamolleti4599/mongospring:latest
```

### Purpose

Downloads latest Docker image from Docker Hub.

---

# Stop Existing Container

```bash
docker stop myspringcontainer || true
```

### Purpose

Stops old container if running.

## || true

Prevents pipeline failure if container does not exist.

---

# Remove Existing Container

```bash
docker rm myspringcontainer || true
```

### Purpose

Removes old container.

---

# Run New Container

```bash
docker run -d \
--name myspringcontainer \
-p 8080:8080 \
--restart unless-stopped \
satyamolleti4599/mongospring:latest
```

### Purpose

Deploys latest Docker image.

---

# --restart unless-stopped

### Purpose

Automatically restarts container.

---

# << EOF

This is called:

```text
Here Document (Heredoc)
```

in Linux shell scripting.

### Purpose

Used to send multiple commands to another command like SSH.

---

# In Your Pipeline

```bash
ssh -o StrictHostKeyChecking=no ubuntu@34.227.48.151 << EOF

docker pull satyamolleti4599/mongospring:latest
docker stop myspringcontainer || true
docker rm myspringcontainer || true
docker run -d --name myspringcontainer -p 8080:8080 satyamolleti4599/mongospring:latest

EOF
```

---

# What Happens?

## Jenkins

* Connects to remote server using SSH.
* Sends all Docker commands to remote server.
* Remote server executes those commands.

---

# Integrating Nexus Repository in Jenkins Pipeline

# Step 1: Create Nexus Container

```bash
docker run -d --name nexus -p 8081:8081 sonatype/nexus3
```

# Step 2: Enable Port 8081

In AWS Security Group:

Add inbound rule

Port:

```text
8081
```

# Step 3: Access Nexus

Open browser:

```text
http://<SERVER-IP>:8081
```

# Step 4: Get Nexus Admin Password

```bash
docker exec -it nexus cat /nexus-data/admin.password
```

Copy password.

# Step 5: Login to Nexus

Default username:

```text
admin
```

Password: Use copied password.

# Step 6: Create Maven Release Repository

Go to:

```text
Settings → Repositories → Create Repository
```

Select:

```text
maven2(hosted)
```

Fill details:

```text
Name: maven-releases
Version: Policy Release
Deployment Policy: Allow Redeploy
```

Click:

```text
Create Repository
```

# Step 7: Create Maven Snapshot Repository

Again create:

```text
maven2(hosted)
```

Fill:

```text
Name: maven-snapshots
Version: Policy Snapshot
Deployment Policy: Allow Redeploy
```

# Step 8: Add Nexus Credentials in Jenkins

Go to:

```text
Manage Jenkins → Credentials
```

Add credentials:

```text
Kind: Username with password
Username: admin
Password: Nexus Password
ID: nexus
Description: nexus-cred
```

# Step 9: Configure pom.xml

Open your project `pom.xml` file.

Add this section before closing:

```xml
</project>
```

```xml
<distributionManagement>

    <repository>
        <id>nexus</id>
        <url>http://<NEXUS-IP>:8081/repository/maven-releases/</url>
    </repository>

    <snapshotRepository>
        <id>nexus</id>
        <url>http://<NEXUS-IP>:8081/repository/maven-snapshots/</url>
    </snapshotRepository>

</distributionManagement>
```

# Configure Maven settings.xml

Open file:

```bash
vi /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/mvn_3.9.15/conf/settings.xml
```

Add inside `<settings>` tag:

```xml
<servers>

    <server>
        <id>nexus</id>
        <username>admin</username>
        <password>admin123</password>
    </server>

</servers>
```

# NOTE

If you already configured Nexus credentials in:

```bash
/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/mvn_3.9.15/conf/settings.xml
```

then Maven will use those credentials automatically during:

```bash
mvn clean deploy
```

No separate Jenkins credentials configuration required for Nexus upload.

# Jenkins Pipeline

```groovy
pipeline {

    agent any

    tools {
        maven 'mvn_3.9.15'
    }

    stages {

        stage('Checkout') {
            steps {

                git branch: 'master',
                url: 'https://github.com/MvvsnmDevops4122/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026.git'

            }
        }

        stage('Build') {
            steps {

                sh 'mvn clean package'

            }
        }

        stage('SonarQube') {
            steps {

                withSonarQubeEnv('sonarQube') {

                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=spring-boot-mongo \
                    -Dsonar.projectName="Spring Boot Mongo Project"
                    '''

                }
            }
        }

        stage('Upload Artifact To Nexus') {

           steps {

           sh 'mvn clean deploy'

                }
           }

        stage('Build Docker Image') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh 'docker build -t satyamolleti4599/mongospring:latest .'

                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker') {

                        sh 'docker push satyamolleti4599/mongospring:latest'

                    }
                }
            }
        }

        stage('Deploy to CD Server') {
            steps {

                sshagent(['cd-ssh']) {

                    sh '''
ssh -o StrictHostKeyChecking=no ubuntu@34.227.48.151 << EOF

docker pull satyamolleti4599/mongospring:latest

docker stop myspringcontainer || true

docker rm myspringcontainer || true

docker run -d \
--name myspringcontainer \
-p 8080:8080 \
--restart unless-stopped \
satyamolleti4599/mongospring:latest

EOF
                    '''

                }
            }
        }
    }
}
```
---

# Push Docker Image to Amazon Web Services ECR and Deploy to CD Server using Jenkins Pipeline

# Step 1: Create ECR Repository

Open AWS Console.

Go to:

```text id="2r1xke"
ECR → Create Repository
```

Repository Name:

```text id="e0lj3o"
mongospring
```

Click:

```text id="i4g4i9"
Create Repository
```

# Step 2: Install AWS CLI on Jenkins Server

SSH into Jenkins server.

Install AWS CLI:

```bash id="3u0z1y"
sudo apt update
```

```bash id="9g4x4v"
sudo apt install awscli -y
```

Verify:

```bash id="n98n3x"
aws --version
```

# Step 3: Configure AWS Credentials on Jenkins Server

Switch to Jenkins user:

```bash id="8cxw2l"
sudo su - jenkins
```

Run:

```bash id="4uaxfy"
aws configure
```

Provide:

```text id="n7r6xg"
AWS Access Key ID
AWS Secret Access Key
Region: us-east-1
Output format: json
```

# Why This Is Required?

ECR is private.

Jenkins needs AWS credentials to:

* Login to ECR
* Push Docker images

Without credentials:

```text id="4nb3f1"
Unable to locate credentials
```

error occurs.

# Step 4: Verify AWS Credentials

Run:

```bash id="wb3m7x"
aws sts get-caller-identity
```

If successful:

* AWS Account ID appears
* IAM user details appear

# Step 5: Install AWS CLI on CD Server

SSH into CD server:

```bash id="uq66j4"
ssh -i key.pem ubuntu@<CD-SERVER-IP>
```

Install AWS CLI:

```bash id="7lqv0q"
sudo apt update
```

```bash id="ny7g3o"
sudo apt install awscli -y
```

Verify:

```bash id="u6l5s1"
aws --version
```

# Step 7: Configure AWS Credentials on CD Server

Run:

```bash id="2k3tws"
aws configure
```

Provide:

```text id="2cfu4x"
Access Key
Secret Key
Region
Output format
```

# Why Required?

CD server must:

* Authenticate to ECR
* Pull Docker image

Without AWS CLI:

```text id="sl0jkq"
aws: command not found
```

Without credentials:

```text id="1l6ogv"
no basic auth credentials
```

# Complete Pipeline

```groovy id="k2o2y7"
pipeline {

    agent any

    tools {
        maven 'mvn_3.9.15'
    }

    stages {

        stage('Checkout') {
            steps {

                git branch: 'master',
                url: 'https://github.com/MvvsnmDevops4122/13_DEVOPS_SPRING_BOOT_MONGO_DOCKER_PROJECT_2026.git'

            }
        }

        stage('Build') {
            steps {

                sh 'mvn clean package'

            }
        }

        stage('SonarQube') {
            steps {

                withSonarQubeEnv('sonarQube') {

                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=spring-boot-mongo \
                    -Dsonar.projectName="Spring Boot Mongo Project"
                    '''

                }
            }
        }

        stage('Upload Artifact To Nexus') {
            steps {

                sh 'mvn clean deploy'

            }
        }

        stage('Build Docker Image') {
            steps {

                sh 'docker build -t satyamolleti4599/mongospring:latest .'

            }
        }

        stage('Push Docker Image To ECR') {
            steps {

                sh '''
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 210856420944.dkr.ecr.us-east-1.amazonaws.com

                docker tag satyamolleti4599/mongospring:latest 210856420944.dkr.ecr.us-east-1.amazonaws.com/mongospring:latest

                docker push 210856420944.dkr.ecr.us-east-1.amazonaws.com/mongospring:latest
                '''

            }
        }

        stage('Deploy to CD Server') {
            steps {

                sshagent(['cd-ssh']) {

                    sh '''
ssh -o StrictHostKeyChecking=no ubuntu@34.227.48.151 << EOF

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 210856420944.dkr.ecr.us-east-1.amazonaws.com

docker pull 210856420944.dkr.ecr.us-east-1.amazonaws.com/mongospring:latest

docker stop myspringcontainer || true

docker rm myspringcontainer || true

docker run -d \
--name myspringcontainer \
-p 8080:8080 \
--restart unless-stopped \
210856420944.dkr.ecr.us-east-1.amazonaws.com/mongospring:latest

EOF
                    '''

                }
            }
        }
    }
}
```

