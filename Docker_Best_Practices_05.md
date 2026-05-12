#  Docker Best Practices

---
## Docker Best Practices

* ✅ Always prefer official and trusted base images when building custom Docker images to improve security, reliability, and stability.

* ✅ Use lightweight base images like Alpine whenever possible to reduce image size.

* ✅ Avoid copying unnecessary files into the image. Use a `.dockerignore` file to exclude unwanted files and folders.

* ✅ Install only the required software and packages to keep the image clean and secure.

* ✅ Reduce the number of image layers by combining related commands into a single `RUN` instruction.

* ✅ Use multi-stage Docker builds to reduce the image size and minimize unnecessary layers in the final image.

* ✅ Run containers using a non-root user to improve container security.

* ✅ Run a shell script daily to remove unused and dangling images.

---

# Docker file 1

## ✅ Nginx Example

```dockerfile id="m6v9q2"
FROM nginx:alpine

COPY index.html /usr/share/nginx/html

# NOTE: create index.html in build context
```

##  Build Image

```bash id="x4k7m1"
docker build -t nginx-satya -f Dockerfile1 .
```

##  Run Container

```bash id="p8v2k5"
docker run -d -p 80:80 --name nginx-cont nginx-satya
```

---

#  Docker file 2

## ✅ Standalone Java Application

```dockerfile id="n5m8x3"
FROM eclipse-temurin:11

WORKDIR /app

COPY target/maven-standalone-application*.jar /app/maven-standalone-application.jar

CMD ["java","-jar","maven-standalone-application.jar"]
```

##  Build Image

```bash id="w2k6p9"
docker build -t mavenapp -f Dockerfile2 .
```

##  Run Container

```bash id="q7v4m1"
docker run --name mavenapp-cont mavenapp
```

---

#  Docker file 3 --> Alpine Example

---

## ✅ Example 1

```dockerfile id="c3m8v5"
FROM alpine:latest

MAINTAINER KKFUNDA <kkeducationblr@gmail.com>

LABEL author="kkfunda"

LABEL email="kkeducationblr@gmail.com"

RUN echo "welcome to kkkfunda"

COPY test.sh test.sh

WORKDIR /opt/app

RUN apt update -y

RUN apt install git wget -y

RUN mkdir -p /opt/app

ENV HOME /opt/app

RUN echo $HOME

CMD ["echo","Hello"]
```

---

## ✅ Example 2

```dockerfile id="f8k2m6"
FROM alpine:latest

MAINTAINER KKFUNDA <kkeducationblr@gmail.com>

LABEL author="kkfunda"

LABEL email="kkeducationblr@gmail.com"

RUN echo "welcome to kkkfunda"

COPY test.sh test.sh

RUN apk update

RUN apk add git wget

CMD ["sh","test.sh"]
```

---

#  Docker file 4 --> Don’t Copy Unnecessary Files

```dockerfile id="v1m7k4"
FROM tomcat:8.0.21-jre8

COPY target/maven-web-application.war /usr/local/tomcat/webapps/maven-web-application.war

COPY . .
```

---

# 📁 What is .dockerignore File?

* The `.dockerignore` file tells Docker which files and directories to ignore during the image build process.

* Mainly used with:

  * `COPY`
  * `ADD`

* Helps:

  * reduce image size
  * improve build speed
  * avoid unnecessary files

---

#  Docker file 5 --> Reduce the Number of Layers

##  Normal Dockerfile

```dockerfile id="t5v8m2"
FROM debian:12.0

MAINTAINER KKFUNDA <kkeducationblr@gmail.com>

LABEL author="kkfunda"

LABEL email="kkeducationblr@gmail.com"

RUN echo "welcome to kkkfunda"

COPY test.sh test.sh

WORKDIR /opt/app

RUN apt update -y

RUN apt install git wget -y

RUN mkdir -p /opt/app

RUN echo "welcome to kk devops"
```

---

## ✅ Optimized Dockerfile (Reduced Layers)

```dockerfile id="k9m4v7"
FROM debian:12.0

MAINTAINER KKFUNDA <kkeducationblr@gmail.com>

LABEL author="kkfunda"

LABEL email="kkeducationblr@gmail.com"

COPY test.sh test.sh

WORKDIR /opt/app

RUN echo "welcome to kkkfunda" && \
    apt update -y && \
    apt install -y git wget && \
    mkdir -p /opt/app && \
    echo "welcome to kk devops"
```

---

#  Docker file 6 --> Try to Run as a Normal User

```dockerfile id="r2m8v1"
# Use the official Tomcat image as the base image
FROM tomcat:9.0

# Create a new group and user
RUN groupadd -r myusergroup && \
    useradd -r -m -g myusergroup -s /bin/bash kkdevops

# Set Tomcat webapps directory as working directory
WORKDIR /usr/local/tomcat/webapps

# Copy WAR file into Tomcat webapps directory
COPY target/maven-web-application.war .

# Change ownership of Tomcat directories to the new user
RUN chown -R kkdevops:myusergroup /usr/local/tomcat && \
    chmod -R 755 /usr/local/tomcat

# Switch to non-root user
USER kkdevops

# Expose Tomcat port
EXPOSE 8080

# Start Tomcat server
CMD ["catalina.sh", "run"]
```

---

## 🔹 Explanation

```dockerfile id="p4m7v2"
RUN groupadd -r myusergroup && \
    useradd -r -m -g myusergroup -s /bin/bash kkdevops
```

### ✅ Creates New Group

```bash id="w8m1k5"
myusergroup
```

### ✅ Creates User

```bash id="u3v9m6"
kkdevops
```

### Options Meaning

| Option | Meaning               |
| ------ | --------------------- |
| `-r`   | system user           |
| `-m`   | create home directory |
| `-g`   | assign group          |
| `-s`   | set shell             |

---

## ✅ Switch Back to Root User

```dockerfile id="g6k2m8"
FROM test3:latest

USER root
```

---

#  Docker file 7 --> Normal Dockerfile

```dockerfile id="y2m5v8"
FROM tomcat:9.0.100

# Install Maven
RUN apt update && apt install maven -y

# Set working directory for building the app
WORKDIR /app

# Copy all files from local source to /app inside the container
COPY . .

# Build the Maven project
RUN mvn clean package

# Change working directory to Tomcat webapps folder
WORKDIR /usr/local/tomcat/webapps

# Copy the built WAR file to Tomcat's webapps directory
RUN cp /app/target/maven-web-app*.war /usr/local/tomcat/webapps/maven-web-app.war
```

---

# Multi-stage Dockerfile using Maven

---

## ✅ Multi-stage Build Definition

* Multi-stage Docker build means using multiple `FROM` instructions in a Dockerfile to separate the build stage and runtime stage.

* Helps:

  * reduce image size
  * remove unnecessary build tools
  * improve security

---

## ✅ Multi-stage Dockerfile

```dockerfile id="d8v3m7"
# Stage 1 - Build Stage
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

COPY . .

RUN mvn clean package

# Stage 2 - Runtime Stage
FROM tomcat:9.0.100

COPY --from=builder /app/target/maven-web-application.war /usr/local/tomcat/webapps/maven-web-application.war
```

---

## ✅ Benefits of Multi-stage Build

* Final image becomes smaller.

* Maven and source code are not included in final image.

* Only WAR/JAR file is copied to runtime image.

* Improves security and performance.

---
