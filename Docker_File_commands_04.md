# Docker File Commands

🔹 A Dockerfile is a blueprint that contains a set of instructions to build a Docker image.

🔹 It decribes all the steps, base image, files, dependencies, configuration, environment settings, and commands needed to package and run your application inside a Docker container.

🔹 Each instruction creates a new layer in the Docker image, and Docker executes these instructions sequentially from top to bottom.

🔹 These includes instructions like `FROM`, `COPY`, `ADD`, `RUN`, `CMD`, `ENTRYPOINT`, `ARG`, `ENV`, `WORKDIR`, `EXPOSE`, `VOLUME`, etc...

🔹 Instructions are not case-sensitive, but use uppercase for better readability.

EX: `FROM = from`

🔹 Default name of docker file is `Dockerfile`

Example for Dockerfile:

```dockerfile
FROM tomcat:9.0.100
COPY target/maven-web-application.war /usr/local/tomcat/webapps/maven-web-application.war
RUN echo "hello kk funda"
```

# 1️. Base Image – FROM

🔹 Syntax:

```dockerfile
FROM <image>:<tag>
```

🔹 Description:

* The FROM instruction specifies the base image used to build your Docker image.

* It is the first instruction in almost every Dockerfile (except in multi-stage builds).

* It defines the starting layer of the Docker image, on top of which all other layers are added.

* Common base images include operating system images like Ubuntu or Alpine, and language-specific images like Java or Node.js.

Example:

```dockerfile
FROM ubuntu:20.04
```

# 2️. Maintainer (Optional)

🔹 Syntax:

```dockerfile
MAINTAINER <name> <email>
```

🔹 Description:

* The MAINTAINER instruction specifies the author or maintainer of the Docker image.

* It acts as metadata to show who built or maintains the image.

* This instruction is deprecated and has been replaced by the LABEL instruction.

Example:

```dockerfile
MAINTAINER KK EDUCATION <kkeducationblr@gmail.com>
```

# 3. LABEL

🔹 Syntax:

```dockerfile
LABEL <key>=<value>
```

🔹 Description:

* The LABEL instruction is used to add metadata to a Docker image as key-value pairs.

* This metadata can include:

  * Image version

  * Author or maintainer information

  * Description or purpose

  * Any custom tags

Examples:

```dockerfile
LABEL version="1.0" description="A sample image"

LABEL company="KK EDUCATION"

LABEL mail="kkeducationblr@gmail.com"

LABEL version="1.0" description="A sample image" company="KK EDUCATION"
```

# 4. COPY – Copy Files from Host to Image

🔹 Syntax:

```dockerfile
COPY <source> <destination>
```

🔹 Description:

* The COPY instruction is used to copy files or directories from the host machine into the Docker image while building the image.

* It works with local files only.

* The source path is relative to the build context (the folder where the Dockerfile is located).

* The destination path is inside the image file system.

Examples:

```dockerfile
COPY target/maven-web-application.war /usr/local/tomcat/webapps/maven-web-application.war

COPY . .                      # Copies everything from current directory to image’s working directory

COPY abc.tar /opt/            # Copies a tar file (but doesn't extract)
```

# 5️. ADD – Add Files (with Extra Features)

🔹 Syntax:

```dockerfile
ADD <source> <destination>
```

🔹 Description:

* The ADD instruction is used to copy files or directories from the host machine or a URL into the Docker image.

* The ADD instruction is similar to COPY, but it offers additional features.

* It can automatically extract compressed files (like `.tar` and `.gz)`into the image.

* It can copy files from a URL directly into the image.

Examples:

```dockerfile
ADD abc.tar /opt/                              # Automatically extracts the tar archive into /opt/
 
ADD https://example.com/file.tar.gz /app/      # Downloads file from the internet
 
ADD ./scripts /opt/scripts                     # Adds a local folder to the image
```
---
```dockerfile
## Dockerfile (Complete ADD Use Case)

FROM ubuntu:20.04

# Set working directory
WORKDIR /demo

# Update and install tools to view files
RUN apt-get update && apt-get install -y curl

# 1. Copy a normal text file
ADD note.txt /demo/

# 2. Add a tar.gz file (it will be automatically extracted)
ADD archive.tar.gz /demo/

# 3. Download a file from the internet
ADD https://github.com/CBO4122/Maven-Web_Application-/blob/master/Jenkinsfile /demo/Jenkinsfile

# Give execute permission to downloaded script
RUN chmod +x /demo/Jenkinsfile

# Final command: List all files and directories

CMD ["bash", "-c", "ls -R /demo && echo 'ADD command demo complete.'"]

```
1. Build the Docker Image: docker build -t demo-add-usecases .

2. Run the Container: docker run --rm demo-add-usecases

Expected Output:
----------------
/demo:
Jenkinsfile
note.txt
webfiles

/demo/webfiles:
index.html
style.css
ADD command demo complete.

---

# 6️. RUN – Execute Commands During Image Build

🔹 Syntax:

```dockerfile
RUN <command>                    # Shell form

RUN ["executable", "arg1"]       # Exec form
```

🔹 Description:

* The RUN instruction is used to execute commands or scripts during the image build process.

* Each RUN instruction creates a new image layer.

* RUN instructions execute only during image creation.

# Common uses:

  * Installing packages

  * Creating directories

  * Setting permissions

  * Configuring environment setups

# Shell Form(default): Runs commands through a shell (for example: /bin/sh -c)

```dockerfile

FROM ubuntu:22.04
LABEL Author="Satya"

# Create directory
RUN mkdir -p /opt/app

# Create user
RUN useradd kkfunda

# Install packages
RUN apt-get update && apt-get install -y git wget tree

# Create file
RUN echo "Welcome to Docker RUN Instruction" > /opt/app/file1.txt

# Create another directory
RUN mkdir /data

CMD ["sleep","500"]
```

# Exec Form (JSON array syntax): Runs commands directly without using a shell.

```dockerfile
 RUN ["apt-get", "install", "-y", "vim"]

 RUN ["/bin/sh", "setup.sh"]

 RUN ["useradd", "kkfunda"]

NOTE: please use shell form for RUN instructions, so we can replace variables easily.
```

# 7️. CMD – Default Command When Container Starts

🔹 Syntax:

```dockerfile
CMD <command>                 # Shell form   

CMD ["executable", "arg1"]    # Exec form
```

🔹 Description:

* The CMD instruction is used to specify the default command or script to run when the container starts.

* You can override CMD at runtime using the Docker CLI (docker run <container> <new_command>).

* Only the last CMD instruction executes if multiple CMD instructions are present.

Examples:

```dockerfile
CMD sh app.sh                      # Shell form

CMD ["java", "-jar", "abc.jar"]    # Exec form
```

## IQ. what is shell form and executable form in docker?

ANS: The RUN, CMD, and ENTRYPOINT instructions can be defined in either shell form or executable form.

When using shell form, the command runs as a "child process" under bash/sh (Shell).

Example: RUN java -jar app.jar

In the background, the above command is executed as:

`/bin/sh -c java -jar app.jar `
[parent process] [child process]

# Note: If you kill the parent process, it will not kill the child process because sometimes the child process is connected to a database.

Example: RUN ["java", "-jar", "app.jar"]

In the background, the above command is executed as:

/bin/java -jar app.jar [parent process]

Note: When using CMD and ENTRYPOINT, the executable form is preferable.

# IQ: Difference Between RUN and CMD

* The RUN instruction is used to execute commands or scripts during the image build process.

* The `CMD` instruction is used to specify the default command or script to run when the container starts.

* We can include multiple RUN instructions in a Dockerfile. All the RUN instructions will be executed in order, from top to bottom, when 
 
  creating an image.

* We can include multiple CMD instructions in a Dockerfile, but only the last one will be executed when the container starts.

RUN sh abc.sh
RUN echo "kkfunda"

CMD echo "one"
CMD echo "two"
CMD echo "three"

## IQ] can you have a more than one CMD in a Dockerfile?

Ans: Yes you have, But only the last one/ recent one in the order will be executed when the container starts.

# 8. ENTRYPOINT

🔹 ENTRYPOINT instruction is used to execute commands or scripts when the container starts.

🔹 You cann't override ENTRYPOINT at runtime using the Docker CLI.

 *  If multiple ENTRYPOINT instructions are used, only the last one takes effect.

 *  When both CMD and ENTRYPOINT are used, CMD values are passed as arguments to ENTRYPOINT..

 👉 Argument: A value or input passed to a command or program while executing it.

🔹 Shell Form:

```dockerfile
ENTRYPOINT java -jar app.jar
```

🔹 Exec Form:

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#  IQ: Difference Between CMD and ENTRYPOINT

   * The CMD instruction is used to specify the default command or script to run when the container starts.

   * You can override CMD at runtime using the Docker CLI (docker run <container> <new_command>).

   * Only the last CMD instruction executes if multiple CMD instructions are present.

  🔹 ENTRYPOINT instruction is used to execute commands or scripts when the container starts.

  🔹 You cann't override ENTRYPOINT at runtime using the Docker CLI.

   *  If multiple ENTRYPOINT instructions are used, only the last one takes effect.

   *  When both CMD and ENTRYPOINT are used, CMD values are passed as arguments to ENTRYPOINT.

 # IQ: Can we use both CMD and ENTRYPOINT in a Dockerfile?


✅ Answer: Yes, we can use both CMD and ENTRYPOINT together in a Dockerfile.But CMD instructions will not be executed if both are 

            present, CMD instructions will be passed as an argument to the ENTRYPOINT.

 Example 1: 

    CMD ["ls"]
    ENTRYPOINT ["echo", "Hello"]

👉 This will execute as: /bin/echo Hello ls

Output: Hello ls

# Real-World Scenario:


If we always want to run sh catalina.sh with a default argument like start, we do:

   CMD ["start"]
   ENTRYPOINT ["sh", "catalina.sh"]

👉 This runs as: /bin/sh catalina.sh start

 Another Example:

    CMD ["pwd"]
    ENTRYPOINT ["echo", "Hello"]

👉 Output: Hello pwd

 # IQ: What is the difference between RUN and docker run?

* The RUN instruction is used to execute commands or scripts during the image build process.

* docker run is a Docker command used to create and start a container. 


# Sample Dockerfile 1
```dockerfile
FROM debian:12.0
MAINTAINER KKFUNDA <kkeducationblr@gmail.com>
LABEL author="kkfunda"
LABEL email="kkeducationblr@gmail.com"

RUN echo "welcome to kkfunda"
RUN apt update -y
RUN apt install git wget tree -y
RUN mkdir -p /opt/app
RUN echo "welcome to kk devops"
```

Build Command: docker build -t firstimage .

# IQ: My Dockerfile contains more layers. How to reduce it?

Answer: You can combine multiple RUN instructions into one using && to reduce layers.(logical  and opertor)

Example:

RUN apt update -y && \
    apt install git wget tree -y && \
    mkdir -p /opt/app && \
    echo "welcome to kkfunda" && \ 
    echo "welcome to kk devops"


# IQ: What is Docker build cache?

* Docker Build Cache is a mechanism that stores previously built image layers during the Docker image build process.

* When the same Dockerfile is built again without any changes, Docker reuses the previously built layers instead of rebuilding them.

* This helps reduce build time and improves build efficiency.

🔁 Example:

docker build -t firstimage .
docker build -t firstimage .   # Second build uses cache, so it's much faster

If you change even one line (e.g., modify a file or a RUN command), Docker will rebuild that layer and all the layers after it.

# Q: How to skip the build cache?

docker build --no-cache -t firstimage1 .

# Q: What happens if we delete all images and build the Dockerfile again?

A: Docker rebuilds everything from scratch, as the cache is lost.

# Dockerfile 2

```dockerfile
FROM debian:12.0
RUN mkdir -p /opt/application
RUN echo "welcome to kk devops"
CMD ["date"]

docker build -t firstimage1 .
docker run --name firstc1 firstimage1
docker start firstc1
docker logs firstc1
```

# Dockerfile 3
```dockerfile
CMD ["date"]
CMD ["echo","java"]
```
👉 Second CMD overrides the first one.

# Dockerfile 4
```
CMD sh test.sh
```
Error: sh: 0: cannot open test.sh: No such file
Because the file is not copied into the image.

# Dockerfile 5
```dockerfile
COPY test.sh test.sh
CMD sh test.sh
```

✅ No error. File is present in the image.

# Dockerfile 6
```dockerfile
ENTRYPOINT ["ls","-lrth"]

docker build -t imageone .
docker run imageone
docker run imageone date
```
# Dockerfile 7

Q: Can we have multiple ENTRYPOINTs?
A: ❌ No. Only the last ENTRYPOINT is used.

ENTRYPOINT ["ls","-lrth"]
ENTRYPOINT ["date"]   # This one will override the previous one

# Dockerfile 8

Q: Can we use both CMD and ENTRYPOINT?
A: ✅ Yes. CMD provides default arguments to the ENTRYPOINT.

CMD ["pwd"]
ENTRYPOINT ["echo","Welcome"]

 Result: echo Welcome pwd

# 9. ARG Instruction

* The ARG instruction is used to define variables that are available only during the Docker image build process. 

* It is mainly used to pass values at build time using the docker build --build-arg option.

* ARG is the only instruction that can be used before the FROM instruction in a Dockerfile.

* Multiple ARG variables can be defined in a Dockerfile and used in other instructions during the image build process.

Example :
```dockerfile
ARG baseImageTag
FROM debian:$baseImageTag
MAINTAINER KKFUNDA <kkeducationblr@gmail.com>
LABEL author="kkfunda"
LABEL email="kkeducationblr@gmail.com"
RUN echo "welcome to kkkfunda"
COPY test.sh test.sh
RUN apt update -y
RUN apt install git wget -y
RUN mkdir -p /opt/application
RUN echo "welcome to kk devops"
```
docker build -t <imageName> --build-arg  baseImageTag=12.0 .

docker build -t <imageName> --build-arg  baseImageTag=11.0 .


Ex: 
```dockerfile
ARG baseImageTag=latest
FROM debian:$baseImageTag
```
# 10. ENV Instruction

* The ENV instruction is used to set environment variables in a Dockerfile.

* ENV variables are available during both the image build process and container runtime.

* ENV variables persist across all Docker image layers and can be used in subsequent instructions.

* ENV is commonly used for application configuration, path settings, Java/Tomcat paths, and environment-specific values.

ENV HOME=/usr/local/tomcat/webapps/
COPY target/web-app.war $HOME
RUN echo $HOME

EX:
    ```dockerfile
     ENV CATALINE_HOME /usr/local/tomcat
     ENV JAVA_HOME /usr/bin/jdk8
    ```

EX:
```dockerfile
FROM debian:12.0
MAINTAINER KKFUNDA <kkeducationblr@gmail.com>
LABEL author="kkfunda"
LABEL email="kkeducationblr@gmail.com"
RUN echo "welcome to kkkfunda"
COPY test.sh test.sh
RUN apt update -y
RUN apt install git wget -y
RUN mkdir -p /opt/app
    
ENV HOME /opt/app
RUN echo $HOME

RUN echo "welcome to kk devops"
```

docker build -t imageone .

docker inspect imageone --> you will the env variables

# 11. WORKDIR Instruction

* The WORKDIR instruction is used to set the current working directory inside the container.

* It is similar to the cd command in Linux.

* If the directory does not exist, Docker automatically creates it.

* WORKDIR helps avoid writing full paths in every instruction.

* We can define multiple WORKDIR instructions to create nested directories.

EX: WORKDIR /usr/local/tomcat


 
EX:
```dockerfile
# Base image with Tomcat
FROM tomcat:9.0

# Set working directory to Tomcat's webapps folder
WORKDIR /usr/local/tomcat/webapps/myapp

# If '/usr/local/tomcat/webapps/myapp' does not exist, Docker creates it.
# If it already exists (e.g., from base image), Docker just enters it.

# Copy the WAR file into the working directory (now inside /webapps/myapp)
COPY target/web-app.war .

# You can run installation or setup commands here if needed
# RUN some-setup.sh

# Start Tomcat (already default, but added for clarity)
CMD ["catalina.sh", "run"]
```

EX:
```dockerfile
FROM debian:12.0
MAINTAINER KKFUNDA <kkeducationblr@gmail.com>
LABEL author="kkfunda"
LABEL email="kkeducationblr@gmail.com"
RUN echo "welcome to kkkfunda"
COPY test.sh test.sh
RUN mkdir -p /opt/app
WORKDIR /opt/app
RUN apt update -y
RUN apt install git wget -y
RUN mkdir -p /opt/test
WORKDIR /opt/test
RUN echo "welcome to kk devops"
COPY abc.txt abc.txt
```
# 12. USER Instruction

* The USER instruction is used to set the user for the container or image.

* After setting USER, all subsequent instructions and container processes run with that user.

* USER helps improve container security by avoiding running applications as the root user.

```dockerfile
FROM ubuntu:22.04
#Create user
RUN useradd -ms /bin/bash kkfunda
# Switch user
USER Satya
CMD ["whoami"]
Build Image: docker build -t userimg .
Run Container : docker run userimg
```

Output: Satya


# 13. EXPOSE

* The EXPOSE instruction is used to specify which port the container listens on during runtime.

EX: EXPOSE 8080


COPY abc.py /opt/abc.py [or] RUN cp abc.py /opt/abc.py


#  IQ: What is Docker Build Cache?

* Docker Build Cache stores previously built image layers.

* When the same Dockerfile is built again without any changes, Docker reuses the previously built layers instead of rebuilding them.

* This makes the Docker image build process faster.

Example:

```bash
docker build -t firstimage .
docker build -t firstimage .
```

#  IQ: How to Skip Docker Build Cache?

```bash
docker build --no-cache -t firstimage .
```

#  IQ: What Happens If We Delete All Images?

* Docker rebuilds everything from scratch because cache layers are removed.

#  Dockerfile Example

```dockerfile
FROM debian:12.0

LABEL author="prasanth"
LABEL email="abc@ibm.com"

RUN echo "welcome to kkfunda"
RUN echo "Hello java"

COPY abc.txt /opt

RUN echo "NELLORE"

RUN apt update -y

RUN apt install git -y

RUN echo "hello Devops"
RUN echo "Welcome to KK FUNDA"

CMD echo "Hello cmd inst"
```

#  IQ: Difference Between RUN and docker run

* The RUN instruction executes commands during image build process.

* `docker run` command creates and starts a container.

#  IQ: Can We Use Multiple ENTRYPOINT Instructions?

* No. Only the last ENTRYPOINT instruction takes effect.

Example:

```dockerfile
ENTRYPOINT ["ls","-lrth"]
ENTRYPOINT ["date"]
```

Here `date` overrides the previous ENTRYPOINT.

# IQ: Can We Use Both CMD and ENTRYPOINT?

* Yes. CMD provides default arguments to ENTRYPOINT.

Example:

```dockerfile
CMD ["pwd"]
ENTRYPOINT ["echo","Welcome"]
```

Output:

```bash
Welcome pwd
```
