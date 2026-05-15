# DOCKER ECR

## What is Docker ECR?

Docker ECR means:

```text
Amazon Elastic Container Registry (ECR)
```

* Amazon ECR is an AWS service used to securely store and manage Docker images in the cloud.
* It is a fully managed Docker image repository provided by AWS.
* ECR helps DevOps engineers push, store, and pull Docker images easily.

---

# Understanding ECR

Think of it like:

| Service    | Purpose                      |
| ---------- | ---------------------------- |
| Docker Hub | Public Image Repository      |
| Amazon ECR | Private AWS Image Repository |

---

# Push Docker Image to AWS ECR

## Step 1: Install unzip

```bash
sudo apt update
```

```bash
sudo apt install unzip -y
```

---

# Step 2: Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

```bash
unzip awscliv2.zip
```

```bash
sudo ./aws/install
```

## Check AWS CLI Version

```bash
aws --version
```

---

# Step 3: Create AWS Access Key

## AWS Console Navigation

```text
Profile → Security Credentials → Access Keys
```

Create:

* Access Key ID
* Secret Access Key

Download or copy them safely.

---

# Step 4: Configure AWS Credentials

Run:

```bash
aws configure
```

Enter:

* AWS Access Key ID
* AWS Secret Access Key
* Default Region Name
* Default Output Format

## Example

```text
Region: eu-north-1
Output format: json
```

---

# Step 5: Create ECR Repository

## AWS Console Steps

1. Open AWS Console
2. Search:

```text
ECR
```

3. Open:

```text
Elastic Container Registry
```

4. Click:

```text
Create Repository
```

5. Enter Repository Name

Example:

```text
kkfunda
```

6. Click:

```text
Create
```

---

# Step 6: View Push Commands

Open Repository:

```text
kkfunda
```

Click:

```text
View Push Commands
```

AWS automatically provides:

* Login Command
* Tag Command
* Push Command

---

# Step 7: Authenticate Docker with ECR

```bash
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.eu-north-1.amazonaws.com
```

## Example

```bash
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 381491819887.dkr.ecr.eu-north-1.amazonaws.com
```

## Successful Output

```text
Login Succeeded
```

---

# Step 8: Tag Docker Image

## Check Docker Images

```bash
docker images
```

## Example Existing Image

```text
springimage:latest
```

## Tag Docker Image

```bash
docker tag springimage:latest 381491819887.dkr.ecr.eu-north-1.amazonaws.com/kkfunda:latest
```

## Verify Tagged Image

```bash
docker images
```

---

# Step 9: Push Image to ECR

```bash
docker push 381491819887.dkr.ecr.eu-north-1.amazonaws.com/kkfunda:latest
```

## Successful Output

```text
Pushed
```

Image will now appear in ECR repository.

---

# Step 10: Pull Image from ECR

## Remove Local Image

```bash
docker rmi 381491819887.dkr.ecr.eu-north-1.amazonaws.com/kkfunda:latest
```

## Pull Image Again

```bash
docker pull 381491819887.dkr.ecr.eu-north-1.amazonaws.com/kkfunda:latest
```

## Verify Pulled Image

```bash
docker images
```

---

# Complete Flow

```text
Docker Build
      ↓
Install AWS CLI
      ↓
Configure AWS Credentials
      ↓
Create ECR Repository
      ↓
Authenticate Docker with ECR
      ↓
Tag Docker Image
      ↓
Push Image to ECR
      ↓
Pull Image from ECR
```

---

# Important Commands Summary

| Command         | Purpose                   |
| --------------- | ------------------------- |
| `aws configure` | Configure AWS Credentials |
| `aws --version` | Check AWS CLI Version     |
| `docker images` | List Docker Images        |
| `docker tag`    | Tag Docker Image          |
| `docker push`   | Push Image to ECR         |
| `docker pull`   | Pull Image from ECR       |
| `docker rmi`    | Remove Local Image        |

---

# Important Notes

* ECR is a private Docker image repository service provided by AWS.
* Docker must authenticate with ECR before push or pull operations.
* Repository URI is required while tagging images.
* AWS CLI must be installed and configured.
* ECR repositories store Docker images securely inside AWS.

---

# Real-Time DevOps Flow

## Step 1

Developer builds Docker image.

```bash
docker build -t springimage .
```

## Step 2

DevOps Engineer pushes image to ECR.

```bash
docker push <ecr-repository-uri>
```

## Step 3

Kubernetes / ECS / Docker Server pulls image from ECR.

```bash
docker pull <ecr-repository-uri>
```

---

# Interview Questions

## What is Amazon ECR?

Amazon ECR is a fully managed AWS Docker image repository used to store, manage, push, and pull container images securely.

---

## Why do we use ECR?

We use ECR to:

* Store Docker images securely
* Integrate with AWS services
* Manage private repositories
* Push and pull images easily

---

## What is the difference between Docker Hub and ECR?

| Docker Hub           | Amazon ECR              |
| -------------------- | ----------------------- |
| Public Repository    | Private AWS Repository  |
| External Service     | AWS Managed Service     |
| Public Image Sharing | Secure Enterprise Usage |

---

## Why do we tag Docker images before pushing to ECR?

Docker images must be tagged with the ECR repository URI before pushing.

Example:

```bash
docker tag springimage:latest 381491819887.dkr.ecr.eu-north-1.amazonaws.com/kkfunda:latest
```

---

## What is the purpose of aws configure?

`aws configure` stores AWS credentials locally so AWS CLI can communicate with AWS services.

---

## Which AWS service stores Docker images?

```text
Amazon Elastic Container Registry (ECR)
```

---

# Summary

Amazon ECR is used to:

* Store Docker images securely
* Push Docker images to AWS
* Pull Docker images from AWS
* Integrate with ECS, EKS, and Kubernetes
* Manage private Docker repositories
