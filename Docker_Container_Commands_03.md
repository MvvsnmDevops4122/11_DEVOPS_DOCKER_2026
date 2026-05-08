#  Docker Container Commands


##  1. Creating a Container

* `docker create`  : It will create a container, But it will not start the container(process)

* `docker run`     : It will create a container and it will start the container(process)

EX: `docker create -p 8080:8080 --name ONE satyamolleti4599/maven-web-app:1.0.0`

ONE == container name , satyamolleti4599/maven-web-app:1.0.0 == images name 

* `docker start ONE`  doc

* `docker run -d -p 8081:8080 --name TWO satyamolleti4599/maven-web-app:1.0.0`

-d == Detached mode  
-p 8081:8080 == Port mapping , (8081 need change host port to 8081 when you are using same container image)

---

##  2. List Containers

* `docker ps` or `docker container ls`           # Running containers

* `docker ps -a` or `docker container ls -a`     # All containers (running + stopped)

* `docker ps -q`                               # Display the running container ids

* `docker ps -aq`                              # Display the running and stopped container ids


---

##  3. Start /  Stop /  Restart /  Pause /  Unpause Container

* `docker start <cid/name>`     EX: docker start e226b7311abc

* `docker stop <cid/name>`      EX: docker stop e226b7311abc

* `docker restart <cid/name>`   EX: docker restart 8c75e27ec677

* `docker pause <cid/name>`     EX: docker pause e226b7311abc

* `docker unpause <cid/name>`   EX: docker unpause e226b7311abc

 
* How to pause all running containers  : `docker pause $(docker ps -aq)`

* How to unpause all paused containers : `docker unpause $(docker ps --filter status="paused" -q)`

* How to delete only paused containers : `docker rm -f $(docker ps --filter status="paused" -q)`


---

##  4. Remove/delete Containers 

* `docker rm <cid>`                       # Remove stopped container

* `docker rm -f <cid>`                    # Force remove running container

* `docker rm -f $(docker ps -q)`          # Remove all running containers

* `docker rm -f $(docker ps -aq)`         # Remove all containers(stoped and runing)

* `docker container prune`                # Remove stopped containers only

EX:
  
* Remove all stopped containers:    `docker rm -f $(docker ps --filter status="exited" -q)`

--filter status="exited"   → Targets only stopped containers  
-q    → Returns only container IDs  
-f    → Force removal without prompt  

* Remove all running containers: `docker rm -f $(docker ps -q)`


---

##  5. Rename a Container

* `docker rename <old_name> <new_name>`

✅ Example: `docker rename TWO SJVN`


---

##  6. Inspecting Docker Containers

* Use docker inspect to view detailed info about containers in JSON format or extract specific fields.

Basic Inspection:  Get complete configuration of a container

* `docker inspect <container_id or container_name>`

✅ Example: `docker inspect 8c75e27ec677`


* Filter Specific Fields with --format:

1. Get container’s IP address: 

* `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' TWO`  
OR) `docker inspect TWO | grep IPAddress`  
OR `docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' TWO`


2️. Get container’s process ID (PID) on the host:

* `docker inspect --format "{{.State.Pid}}" TWO`


3️. Get IP address and container status:

* `docker inspect --format "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}},{{.State.Status}}" TWO`


4. Get the container’s start time:
 
* `docker inspect --format "{{.State.StartedAt}}" TWO`

* `docker inspect -f "{{.State.StartedAt}}" TWO`


5. Get StartedAt times for all running containers:

* `docker inspect -f "{{.State.StartedAt}}" $(docker ps -q)`


6. Get StartedAt times for all containers (running + stopped):

* `docker inspect -f "{{.State.StartedAt}}" $(docker ps -aq)`


---

##  Dangling Docker Images

* How to View Dangling Images: `docker images -f dangling=true`

* How to View Non-Dangling Images (normal images): `docker images -f dangling=false -q`

* How to Remove Dangling Images: `docker rmi -f $(docker images -f dangling=true -q)`


---

## IQ: docker stop vs docker kill

* `docker stop`: not force kill

* `docker kill`: force kill


---

##  Debugging & Logs

* `docker logs <container_id_or_name>`  

* `docker logs --tail <number_of_lines> <container_name>`  

* `docker logs -f <container_name>`  


---

##  Go Inside the Container (Interactive Shell)

* `docker exec -it <cid/name> /bin/bash`

* `exit`


---

## View Files Inside Without Entering Container

* `docker exec <container_name> ls`

 Example: `docker exec javawebapp ls /usr/local/tomcat`

---

##  View Running Processes Inside a Container

* `docker top <cid/cname>`

---

## Monitor Resource Usage (CPU, Memory)

* `docker stats <cid/cname>`

---

##  IQ: Custom Memory Allocation

* `docker run --memory="512m" -d -p 8083:8080 --name ONE satyamolleti4599/maven-web-app:1.0.0`

* `docker run --memory="64m" -d  -p 8084:8080 --name FIVE satyamolleti4599/maven-web-app:1.0.0`

* `docker inspect <cid>`
