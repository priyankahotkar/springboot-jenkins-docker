# Spring Boot + Jenkins + Docker — Demo Project

**Repository:** priyankahotkar/springboot-jenkins-docker

This package contains a minimal Spring Boot application that Jenkins can build, dockerize, and run.
It includes two workflows in the README: (A) Local Jenkins (install on machine) and (B) Jenkins in Docker.

---

## What's included
- `pom.xml` - Maven build for Spring Boot
- `src/main/java/com/example/demo/DemoApplication.java` - simple REST endpoint `/`
- `Dockerfile` - builds a runnable image from `target/*.jar`
- `Jenkinsfile` - pipeline to build, dockerize, run, and optionally push image
- `README.md` - this document

---

## Quick local build & run (manual)
1. Build with Maven (needs Java 17 & Maven):
```
mvn clean package
```
2. Build Docker image:
```
docker build -t priyankahotkar/springboot-jenkins-docker:latest .
```
3. Run container:
```
docker run -d --name jenkins-demo -p 8080:8080 priyankahotkar/springboot-jenkins-docker:latest
```
4. Open: http://localhost:8080/  → should return the greeting text.

---

## A) Exam-friendly: Jenkins installed on local machine (recommended for exam)

### Pre-reqs on Windows (local Jenkins)
- Install JDK 17
- Install Maven
- Install Docker Desktop and ensure Docker is running
- Download & install Jenkins (Windows installer) from https://www.jenkins.io/download/

### Steps
1. Start Jenkins service and open `http://localhost:8080`
2. Create a new Pipeline job:
   - Choose **Pipeline** and set **Pipeline script from SCM**
   - SCM: Git, Repository URL: `https://github.com/priyankahotkar/springboot-jenkins-docker.git`, Branch: `main`
3. Ensure Jenkins has Maven & JDK configured in **Manage Jenkins → Global Tool Configuration** with names used in your jobs if needed.
4. Give Jenkins access to Docker (Docker Desktop running on host; Jenkins runs on host so Docker CLI is accessible).
5. Add Credentials (optional) for Docker Hub if you plan to push images: **Credentials → System → Global** → Add username/password (id: `dockerhub-creds`)
6. Run the job. The pipeline will:
   - Checkout code
   - Run `mvn clean package`
   - Build Docker image and run the container on port 8080
7. Verify at `http://localhost:8080/`

Notes for exam mode: Running Jenkins on local host avoids Docker-in-Docker complexities and is fast to demonstrate.

---

## B) Advanced: Jenkins running inside Docker (CI server in container)

### Pre-reqs
- Docker Desktop on Windows (WSL2 recommended)
- Jenkins running in Docker with access to Docker daemon (mount docker socket)
- A docker network (optional) if you prefer to isolate containers

### Run Jenkins in Docker (Windows PowerShell example)
```
# create a local folder to persist Jenkins data
mkdir C:/Users/PRIYANKA/jenkins_home

docker run -d --name jenkins --restart=on-failure -p 8080:8080 -p 50000:50000   -v "C:/Users/PRIYANKA/jenkins_home:/var/jenkins_home"   -v "//var/run/docker.sock:/var/run/docker.sock"   -v "C:/Program Files/Docker/Docker/resources/bin/docker.exe:/usr/bin/docker"   jenkins/jenkins:lts-jdk17
```

> If you hit socket or path issues on Windows, consider installing Jenkins directly on host (Option A) for exam/demo; it’s simpler.

### Configure Jenkins (inside container)
1. Open Jenkins UI `http://localhost:8080/`
2. Install suggested plugins and set up admin user
3. Add Docker Hub credentials (optional) in **Credentials** with id `dockerhub-creds`
4. Create Pipeline job (Pipeline script from SCM) pointing to this repo
5. Ensure the Jenkins container can run `docker` (verify `docker ps` inside Jenkins container)

### Notes about Jenkinsfile
- The Jenkinsfile in this repo runs shell commands (`sh`). If your Jenkins agent is Windows-based or uses PowerShell, adapt commands accordingly.
- The pipeline builds the project with Maven, builds Docker image, runs container, and optionally pushes to Docker Hub when `DOCKER_PUSH=true` and `dockerhub-creds` is configured.

---

## Troubleshooting — quick checklist
- If `docker: command not found` inside Jenkins container → mount Docker socket and docker.exe as explained, or run Jenkins on host.
- If build fails with `Tool type "maven" does not have an install of "Maven3" configured` → configure Maven in Jenkins with the same label or use wrapper commands.
- If container port conflict → stop existing container or change host port binding

---

## Push to Docker Hub (optional)
1. Create a Docker Hub repo `priyankahotkar/jenkins-dockerize`
2. In Jenkins, add Docker Hub credentials with id `dockerhub-creds`
3. In Jenkins job, set environment variable `DOCKER_PUSH=true` (or update pipeline to pass it)
4. Pipeline will login and push the image to your Docker Hub repo

---

## Notes
- This project targets Java 17 and Spring Boot 3.1.x. Adjust `pom.xml` `java.version` if needed.
- For exam/demo, pick Option A (Local Jenkins) to save time and avoid WSL/socket issues.

---
Good luck — if you want, I can push this ZIP contents to your GitHub repo or provide a `.zip` you can download now.
