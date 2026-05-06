#  What Is Docker?

**Docker is an open-source containerization platform.**

* It allows developers to package applications and their dependencies into a standardized unit called a **container**.

* Developers can **build, run, and manage applications** using Docker.

* Applications can run across different environments **without any compatibility issues**.

* Docker containers are **lightweight, portable, and faster** than virtual machines.

* Containers share the **host system’s kernel**, unlike virtual machines which need a separate OS.

* Docker is ideal for running applications consistently on **laptop, server, or cloud**.

* Docker is supported on **Linux, Windows, and macOS**.

* **Memory sharing is possible in Docker.**

---

#  Virtual Machine (VM)

* A Virtual Machine (VM) is software that behaves like a **separate physical computer**.

* It runs its own operating system (OS) and applications, but inside a host machine using a hypervisor like VMware, Oracle VirtualBox, or Microsoft Hyper-V.

   🔹 Each VM gets its own dedicated memory (RAM), CPU, and storage, even though it’s all coming from the same physical system.

   🔹 It is heavy weight and Non-portable

   🔹 Memory sharing is not possible.

  <img width="905" height="617" alt="image" src="https://github.com/user-attachments/assets/cccb26e5-3727-416a-be91-ba87abca14ca" />

---

#  Similar Tools to Docker 

*  Podman
*  Rkt (Rocket)
*  LXC (Linux Containers)
*  Singularity

---

#  Docker Editions

🔹 **Community Edition (CE)** → Open-source and free to use.

🔹 **Enterprise Edition (EE)** → Commercial version with advanced security and support features

---

#  Method 1: Installing Docker on Ubuntu EC2

---

##  Step 1: Launch an EC2 Instance

* 🔸 Choose **Ubuntu 20.04 or later**
* 🔸 Select instance type (e.g., **t2.micro**)
* 🔸 Connect using **SSH**

---

##  Step 2: Install Docker

```bash
sudo apt update -y   # Before installing Docker, always update your package list
sudo apt install docker.io -y
```

---

##  Step 3: Verify Docker Installation

```bash
docker -v       # Shows Docker version
docker info     # May show 'permission denied' initially
```

---

##  Step 4: Check Docker Service Status

```bash
ps -ef | grep -i "docker"
sudo service docker status
```

---

##  Step 5: Add Current User to Docker Group

```bash
sudo usermod -aG docker ubuntu
```

---

##  Step 6: Run Docker Commands Without sudo

```bash
docker ps
docker info
docker images
```

---

#  Method 2: Using Docker Official Script

```bash
sudo apt update -y
sudo curl -fsSL https://get.docker.com -o install-docker.sh
sudo sh install-docker.sh
```

---

#  Method 3: Direct Install Command

```bash
sudo apt update -y
sudo curl -fsSL https://get.docker.com | /bin/bash
```

---
