# AWS Ubuntu Server Setup with Dockerised Spring Boot Web Application

## 📝 Overview

This project demonstrates how to deploy a simple web application using Docker on an Ubuntu server hosted on AWS EC2. It includes setting up secure SSH access, firewall rules via AWS Security Groups, and pushing the Docker image to DockerHub for portability and version control.

---

## 📋 Requirements

- **Ubuntu Server**: 22.04 LTS
- **EC2 Instance Type**: `t2.micro`
    - vCPU: 1
    - Memory: 1 GB
    - Free Tier Eligible
- **DockerHub Account**
- **AWS Account with EC2 Permissions**
- **Spring Boot Web Application (packaged in Docker)**

---

## 🔐 SSH & Firewall Configuration

### SSH Access

| Setting | Configuration |
| --- | --- |
| Allow SSH (port 22) | Only from **your current IP address** (use `My IP` in AWS Security Group) |
| Authentication method | **SSH key pair only**, no password authentication |
| User account | Use a non-root user like `ubuntu`; disable root login |

> ✅ This limits exposure to brute-force attacks.
>

---

### AWS Security Group Rules

| Rule Name | Port | Protocol | Source | Purpose |
| --- | --- | --- | --- | --- |
| SSH | 22 | TCP | Your IP | Admin access only |
| HTTP | 80 | TCP | 0.0.0.0/0 | Web app access for all |
| (Optional) HTTPS | 443 | TCP | 0.0.0.0/0 | For SSL-enabled deployments |

> 🔒 Never expose SSH (port 22) to 0.0.0.0/0.
>

---

## 🧱 Project Structure

```
hellospring/
├── Dockerfile
├── target/
│   └── hellospring-1.0.0-SNAPSHOT.jar
├── src/ 
└── README.md

```

---

## 🐳 Docker Setup

### Dockerfile

```powershell
# Use an official OpenJDK base image
FROM openjdk:17-jdk-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the built jar into the container
COPY target/hellospring-1.0.0-SNAPSHOT.jar app.jar

# Expose port 8080
EXPOSE 8080

# Command to run the application
ENTRYPOINT ["java", "-jar", "app.jar"]

```

### Docker Build for multiple platforms

Make sure you:

- Have **Docker 20.10+** installed
- Have **Buildx** enabled (comes with recent Docker versions)

```bash
docker --version
docker buildx version

#This creates a new builder that supports cross-platform builds.
docker buildx create --use --name multiarch-builder
docker buildx inspect --bootstrap

```

```bash
#Package the java web application
./mvnw clean package

#Build with multiple platforms, tag and push to the DockerHub
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t admin4docker/hellospring:1.0.0 \
  --push .

```

![Screenshot 2568-06-20 at 12.10.21 PM.png](screenshots/Screenshot_2568-06-20_at_12.10.21_PM.png)

---

## ☁️ EC2 Server Deployment Steps

1. Launch EC2 instance (Ubuntu 22.04, `t2.micro`)

   ![Screenshot 2568-06-20 at 10.25.04 AM.png](screenshots/Screenshot_2568-06-20_at_10.25.04_AM.png)

2. Create security group with the folowing inbound rules and use it in EC2 instance.

![Screenshot 2568-06-20 at 12.33.37 PM.png](screenshots/Screenshot_2568-06-20_at_12.33.37_PM.png)

1. SSH into the server:

    ```bash
    ssh -i my-key.pem ubuntu@<ec2-public-ip>
    
    ```

2. Install Docker:

    ```bash
    sudo apt update
    sudo apt install docker.io -y
    
    ```

3. Pull Docker image:

    ```bash
    sudo docker pull <your-docker-hub-username>/hellospring:1.0.0
    ```

4. Run container:

    ```bash
    sudo docker run -d -p 80:8080 <your-docker-hub-username>/hellospring:1.0.0
    
    ```

   ![Screenshot 2568-06-20 at 12.44.04 PM.png](screenshots/Screenshot_2568-06-20_at_12.44.04_PM.png)

5. Accessing from the browser.

![Screenshot 2568-06-20 at 1.13.56 PM.png](screenshots/Screenshot_2568-06-20_at_1.13.56_PM.png)

## 📄 Documentation & Maintenance

### SSH Access

```bash
ssh -i my-key.pem ubuntu@<ec2-ip>

```

### Restart the App

```bash
docker ps               # get container ID
docker restart <ID>     # restart container

```

### Update the App

1. Rebuild and push updated image locally
2. Pull updated image on EC2
3. Stop and remove old container
4. Run new container

---

## 📌 Key Takeaways

### 🔐 Security

- Key-based SSH only, limited to one IP
- No password login, no open ports except 80 (and optionally 443)

### 🌐 Network Exposure

- HTTP and HTTPS only to public
- All other services blocked unless explicitly required

### 📦 Deployment

- Docker-based deployment ensures consistency
- Easily replicable across environments
- Portable via DockerHub

---

## 🛠️ Tech Stack

| Tool | Purpose |
| --- | --- |
| AWS EC2 | Hosting the Ubuntu server |
| Ubuntu 22.04 | OS |
| Docker | Containerization |
| Spring Boot | Web application backend |
| Docker Hub | Image registry |
| Git & GitHub | Version control & documentation |

---

## 📚 Future Improvements

- Add CI/CD pipeline (GitHub Actions)
- Add Nginx reverse proxy with HTTPS
- Auto-restart container using systemd or Docker restart policies

---

## 👤 Author

Chaw Theingi

[GitHub](https://github.com/chawtg) • [Docker Hub](https://hub.docker.com/repositories/admin4docker)
