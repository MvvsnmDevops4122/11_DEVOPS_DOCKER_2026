#  Docker Networks

## Docker Networks

- Docker networks are used to enable communication between containers.

- If both containers are on the same network, we can establish a connection between them.

- If you do not specify a network when creating a container, it will be created in the **default bridge network**.

- By default, Docker provides three types of networks:

```text
1. bridge (default)
2. host
3. none
```

---

## How to List Docker Networks?

```bash
docker network ls
```

---

## How to Check Which Network a Container is Using?

```bash
docker inspect <cid>/<cname>
```

Example:

```bash
docker inspect cont-one
```

---

##  How to Check If Two Containers Are Connected (Default Bridge Network)

### Step 0: Create Two Containers

```bash
docker build -t image1 .

docker run -d -p 8080:8080 --name cont-one image1
docker run -d -p 8081:8080 --name cont-two image1
```

---

### Step 1: Get the IP Address of Second Container

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cont-two
```
Suppose the result is: 172.17.0.3

Example Output:

```text
172.17.0.3
```

---

### Step 2: Enter the First Container

```bash
docker exec -it cont-one /bin/bash
```

---

### Step 3: Install Ping Utility (Only Once)

```bash
apt update -y && apt install iputils-ping -y
```

---

### Step 4: Ping the Second Container by IP

```bash
ping 172.17.0.3
```

---

### Step 5: Test Port Connectivity

```bash
curl -v telnet://172.17.0.3:8080
```

✅ If ping and curl succeed, the containers are able to communicate.

---

##  Disadvantages of Default Bridge Network

- In the default bridge network, containers can communicate only using IP addresses. Container names are not supported for communication.

- If a container is restarted, Docker may assign a new IP address to the container, so the previous IP address is not guaranteed to remain the same.

---

##  Difference Between Default Bridge and Custom Bridge Networks in Docker

- In the default bridge network, you can communicate with another container only by using its IP address, not the container name.

- In a custom bridge network, you can communicate with another container using either its IP address or container name.

---

| Feature | Default Bridge Network | Custom Bridge Network |
|----------|-------------------------|------------------------|
| Communication | Only via IP address | Via IP address and container name |
| DNS Resolution | ❌ Not Supported | ✅ Supported |
| Container Name Access | ❌ Not Possible | ✅ Possible |
| Use Case | Basic setup/testing | Recommended for real applications |
| IP Stability | ❌ Not Guaranteed | ✅ Better Management |

---

## Custom Bridge Network : Better Approach

### Step 0: Create Custom Bridge Network

```bash
docker network create -d bridge jionetwork
```

### Explanation

```text
bridge      -> Driver Name
jionetwork  -> Custom Network Name
```

---

### Step 1: Run Containers on Custom Network

```bash
docker run -d -p 8082:8080 --network jionetwork --name cont-four image1

docker run -d -p 8083:8080 --network jionetwork --name cont-five image1
```

---

### Step 2: Get the IP of Second Container

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cont-five
```

Example Output:

```text
172.18.0.3
```

---

### Step 3: Enter the First Container

```bash
docker exec -it cont-four /bin/bash
```

---

### Step 4: Install Ping Utility

```bash
apt update -y && apt install iputils-ping -y
```

---

### Step 5: Ping Second Container

#### Using IP Address

```bash
ping 172.18.0.3
```

#### Using Container Name

```bash
ping cont-five
```

- In custom bridge networks, container names work because Docker provides internal DNS resolution.

---

## How to Remove Unused Networks?

```bash
docker network prune
```

Removes all unused networks.

---

## How to Display Detailed Information About Networks?

```bash
docker network inspect <network-name/network-id>
```

Example:

```bash
docker network inspect jionetwork
```

---

## How to Remove Dangling Images?

```bash
docker image prune
```

---

## How to Remove One or More Networks?

```bash
docker network rm <network-name>
```

Example:

```bash
docker network rm jionetwork
```

---

## How to Add a Container to Another Network?

```bash
docker network connect jio-retail cont-four
```

### Explanation

```text
jio-retail -> Network Name
cont-four  -> Container Name
```

### Verify

```bash
docker inspect jio-retail
docker inspect cont-four
```

---

## How to Disconnect a Container from a Network?

```bash
docker network disconnect jio-retail cont-four

docker inspect jio-retail(inspecting by network)

docker inspect cont-four(inspecting by container)
```

### Verify

```bash
docker inspect jio-retail
docker inspect cont-four
```

---

##  What is Docker Host Network?

* In Docker host networking, the container uses the host machine’s network directly and does not get a separate IP address.

* Since containers share the host network, multiple containers cannot use the same port number at the same time.

---

##  Practical Understanding of Host Network

### Step 1: Run First Container

```bash
docker run -d --network host --name app1 nginx
```

### What Happens Here?

- nginx runs on port `80`
- container uses the host machine network directly

So now:

```text
Server-IP:80 → app1
```

Example:

```text
http://<server-ip>
```

opens nginx page.

---

### Step 2: Run Second Container

```bash
docker run -d --network host --name app2 nginx
```

This container also tries to use:

```text
Server-IP:80
```

But port `80` is already used by `app1`.

So Docker stops `app2`.

---

##  Why Did It Fail?

Because in host networking:

```text
Both containers share the same server network.
```

There is only one:

```text
Port 80
```

on the server.

So:

```text
One Port = One Container
```

in host network mode.

---

##  What is None Network?

- If you create containers in none network, the container will not have an IP address.

- We cannot access these containers from outside, from other containers, or even from the internet.

---

### Q) What is Overlay Network?

Overlay network is used to establish communication between containers running on different Docker hosts/servers.
