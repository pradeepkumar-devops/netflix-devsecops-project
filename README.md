# 🎬 Netflix DevSecOps CI/CD Pipeline

An end-to-end **DevSecOps CI/CD pipeline** for a Netflix-clone web application — built from scratch on **AWS EC2**, using **Jenkins, SonarQube, Trivy, Docker, DockerHub, Prometheus, and Grafana**.

This project simulates a real-world production pipeline: code is checked out from GitHub, scanned for quality and security issues, containerized, pushed to a registry, deployed, and continuously monitored — all automatically.

---

## 📌 Project Overview

| | |
|---|---|
| **Type** | DevSecOps CI/CD Pipeline |
| **Application** | Netflix-clone web app |
| **Cloud** | AWS EC2 (Free Tier) |
| **CI/CD** | Jenkins |
| **Security** | SonarQube (code quality) + Trivy (vulnerability scanning) |
| **Containerization** | Docker + DockerHub |
| **Monitoring** | Prometheus + Node Exporter + Grafana |
| **Automation** | GitHub Webhook (auto-trigger builds) |
| **Pipeline as Code** | Jenkinsfile stored in the repository |

---

## 🏗️ Architecture

```
                         Developer
                             │
                             ▼
                    GitHub Repository
                             │
                (GitHub Webhook triggers build)
                             ▼
                     Jenkins Pipeline
                             │
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
   Git Checkout       SonarQube Scan         Trivy FS Scan
        │                    │                    │
        └────────────────────┴────────────────────┘
                             ▼
                      Docker Build
                             │
                             ▼
                   Trivy Image Scan
                             │
                             ▼
                Push Image to DockerHub
                             │
                             ▼
                 Deploy Docker Container
                             │
                             ▼
                  Netflix Application (Live)
                             │
                             ▼
              Prometheus + Node Exporter
                             │
                             ▼
                   Grafana Dashboard
```

---

## 🛠️ Tech Stack

**Cloud & Infrastructure:** AWS EC2, Elastic IP, Ubuntu Server, Linux Administration
**Version Control:** Git, GitHub, GitHub Webhooks
**CI/CD:** Jenkins (Pipeline as Code via Jenkinsfile)
**Code Quality:** SonarQube
**Security Scanning:** Trivy (Filesystem + Image scans)
**Containerization:** Docker, DockerHub
**Monitoring:** Prometheus, Node Exporter, Grafana

---

## 🚀 Features Implemented

- ✅ AWS EC2 instance with Elastic IP for a stable public endpoint
- ✅ Dockerized Netflix-clone application
- ✅ Jenkins CI/CD pipeline fully automated end-to-end
- ✅ **Pipeline as Code** — Jenkinsfile version-controlled in the repo (not written manually in the Jenkins UI)
- ✅ **GitHub Webhook** — pipeline triggers automatically on every `git push`, no manual "Build Now" needed
- ✅ SonarQube integration for static code analysis
- ✅ Trivy filesystem scan (scans source code/dependencies for vulnerabilities)
- ✅ Trivy image scan (scans the built Docker image before deployment)
- ✅ Automated Docker image build, tag, and push to DockerHub
- ✅ Secure credential management using Jenkins Credentials Store
- ✅ Automated container deployment
- ✅ Real-time infrastructure monitoring with Prometheus + Node Exporter
- ✅ Visual dashboards in Grafana (CPU, memory, disk, network, load)
  

---

## ⚙️ Setup Guide (From Scratch)

## PHASE 1 – AWS Infrastructure Setup
Step 1: Create GitHub Repository First
-Login to GitHub.
-Click: New Repository

-Repository Name:netflix-devsecops-project

#Select:

✓ Public
✓ Add README

#Click:Create Repository

## Step 2: Create AWS Free Tier EC2
->Open AWS Console:EC2 → Launch Instance
->Settings:
->Name:netflix-devsecops-server
->AMI:Ubuntu Server 24.04 LTS
->Instance Type:t2.micro (or) t3.micro   (Free Tier eligible)

## Step 3: Create Key Pair
->Click:Create New Key Pair
->Name:netflix-key
->Type:RSA
 ->Format:.pem
-?Download and save safely.

## Step 4: Configure Storage
->Change:8 GB → 30 GB
->Reason:Jenkins + SonarQube + Docker images need space.

## Step 5: Configure Security Group

| **Type**   | **Port** | **Source**           |
| ---------- | -------: | -------------------- |
| SSH        |       22 | Anywhere (0.0.0.0/0) |
| HTTP       |       80 | Anywhere (0.0.0.0/0) |
| Custom TCP |     8080 | Anywhere (0.0.0.0/0) |
| Custom TCP |     9000 | Anywhere (0.0.0.0/0) |
| Custom TCP |     3000 | Anywhere (0.0.0.0/0) |
| Custom TCP |     9090 | Anywhere (0.0.0.0/0) |

## Step 6: Launch Instance
->Click:Launch Instance
->Wait until:Instance State = Running

## Step 7: Get Public IP
->Copy:Public IPv4 Address
->Example:13.233.xxx.xxx
->Save it.

## Step 8: Connect Using MobXterm
->Open 'MobXterm'.
->Click Session → SSH.
->In Remote host, enter your EC2 Public IP Address.
->Specify the username: 'ubuntu'
->Check Use private key.
->Browse and select your private key file: 'netflix-key.pem'
->Click OK to connect.

"Note: If prompted to accept the server fingerprint, click Accept or Yes."

## Step 9: Verify Connection
Run: 'whoami'
->Expected output: 'ubuntu'
->Run:pwd
->Expected output: '/home/ubuntu'
## Step 10: Update Server
Run: 'sudo apt update'
->Then: 'sudo apt upgrade -y'
->This may take several minutes.

## Step 11: Install Basic Packages
Run:sudo apt install -y git curl wget unzip vim net-tools
- ✅ Verify Git: 'git --version'
- ✅ Verify Curl: 'crl --version'
  
## Step 15: Allocate Elastic IP
->In AWS Console:EC2 → Network & Security → Elastic IPs
->Click:Allocate Elastic IP Address
->Keep defaults and click:
Allocate
## Step 16: Associate Elastic IP
->Select the newly created Elastic IP.
Click:
Actions → Associate Elastic IP Address

### Choose:
Resource Type: Instance
Instance: netflix-devsecops-server
Private IP: Default
Click:Associate
## Step 17: Verify
Go to:EC2 → Instances
### You should now see:
Public IPv4 Address = Elastic IP

###Example:
43.xx.xx.xx
###This IP will remain the same even after restarting the instance.

# PHASE 2 – Docker Installation

## Step 1: Check System Resources

Before installing Docker, check the available system resources.

### Check RAM

```bash
free -h
```

### Check Disk Space

```bash
df -h
```

### Check CPU Cores

```bash
nproc
```

These commands display:

- Available RAM
- Available Disk Space
- Number of CPU Cores

---

# Step 2: Install Docker

Download the official Docker installation script.

```bash
curl -fsSL https://get.docker.com -o install-docker.sh
```

Verify the script has been downloaded.

```bash
ls -l
```

You should see:

```
install-docker.sh
```

Run the installation.

```bash
sudo sh install-docker.sh
```

Wait until the installation completes.

---

# Step 3: Verify Docker Service

Check Docker service status.

```bash
sudo systemctl status docker
```

Expected output:

```
active (running)
```

---

# Step 4: Enable Docker at Boot

Enable Docker to start automatically after every reboot.

```bash
sudo systemctl enable docker
```

Verify:

```bash
sudo systemctl is-enabled docker
```

Expected:

```
enabled
```

---

# Step 5: Allow Ubuntu User to Use Docker

Without this configuration you may encounter the following error:

```
permission denied while trying to connect to docker.sock
```

Add the Ubuntu user to the Docker group.

```bash
sudo usermod -aG docker ubuntu
```

Apply the group changes immediately.

```bash
newgrp docker
```

---

# Step 6: Verify Docker Installation

Check Docker version.

```bash
docker --version
```

Example:

```
Docker version 28.x.x
```

---

# Step 7: Test Docker

Run the Docker Hello World container.

```bash
docker run hello-world
```

Expected output:

```
Hello from Docker!
```

If you see this message, Docker is installed correctly.

---

# PHASE 2.1 – Create Project Directory

Create the workspace.

```bash
mkdir -p ~/projects/netflix-devsecops-project
```

Move into the project directory.

```bash
cd ~/projects/netflix-devsecops-project
```

Verify the current location.

```bash
pwd
```

Expected:

```
/home/ubuntu/projects/netflix-devsecops-project
```

---

# PHASE 2.2 – Initialize Git Repository

Initialize Git.

```bash
git init
```

Configure Git (if needed).

```bash
git config --global user.name "Pradeep Kumar"
git config --global user.email "YOUR_GITHUB_EMAIL"
```

Verify the configuration.

```bash
git config --list
```

---

# PHASE 2.3 – Create Netflix Application Files

Create the required files.

```bash
touch index.html
touch style.css
touch Dockerfile
touch README.md
```

Verify the files.

```bash
ls
```

Expected output:

```
Dockerfile
README.md
index.html
style.css
```

---

# Step 8: Create Netflix Homepage

Open the HTML file.

```bash
vim index.html
```

Press **`i`** to enter Insert Mode.

Paste the following code.

```html
<!DOCTYPE html>
<html>
<head>
    <title>Netflix Clone</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>NETFLIX</h1>
    <h2>Unlimited movies, TV shows and more.</h2>
    <p>DevSecOps Project by Pradeep Kumar</p>
</body>
</html>
```

Save and exit.

```
ESC
:wq
```

---

# Step 9: Create CSS

Open the CSS file.

```bash
vim style.css
```

Press **`i`** and paste the following.

```css
body{
    background:black;
    color:white;
    text-align:center;
    font-family:Arial;
    margin-top:100px;
}

h1{
    color:red;
    font-size:60px;
}
```

Save and exit.

```
ESC
:wq
```

---

# Step 10: Test the Website Locally

Install Nginx.

```bash
sudo apt install nginx -y
```

Copy the website files.

```bash
sudo cp index.html /var/www/html/
sudo cp style.css /var/www/html/
```

Open your browser.

```
http://YOUR_ELASTIC_IP
```

You should see the Netflix homepage.

---

# PHASE 2.4 – Dockerize the Application

Create the Dockerfile.

```bash
vim Dockerfile
```

Press **`i`** and paste:

```dockerfile
FROM nginx:latest

COPY . /usr/share/nginx/html

EXPOSE 80
```

Save and exit.

```
ESC
:wq
```

---

# Step 11: Build Docker Image

Inside the project directory, build the Docker image.

```bash
docker build -t netflix:v1 .
```

Verify the image.

```bash
docker images
```

Expected output:

```
REPOSITORY    TAG
netflix       v1
```

---

# Step 12: Run Docker Container

Since Nginx is already using Port **80**, stop it first.

```bash
sudo systemctl stop nginx
```

Run the Docker container.

```bash
docker run -d --name netflix-container -p 80:80 netflix:v1
```

Verify the running container.

```bash
docker ps
```

Open your browser.

```
http://YOUR_ELASTIC_IP
```

You should now see the Netflix webpage served from the Docker container.

---

# Step 13: Commit the Project to GitHub

Create a `.gitignore` file.

```bash
touch .gitignore
```

Add the following content.

```text
*.log
```

Stage all files.

```bash
git add .
```

Create the first commit.

```bash
git commit -m "Initial Netflix application and Docker setup"
```

Connect your GitHub repository.

```bash
git remote add origin YOUR_REPOSITORY_URL
```

Rename the default branch.

```bash
git branch -M main
```

Push the project.

```bash
git push -u origin main
```

---

# Phase 2 Completion Checklist

Verify the following commands before proceeding.

### Docker Version

```bash
docker --version
```

### Running Containers

```bash
docker ps
```

### Available Docker Images

```bash
docker images
```

### Verify the Application Locally

```bash
curl http://localhost
```

---

# ✅ Phase 2 Completion Checklist

- ✅ Docker installed successfully
- ✅ Docker service running
- ✅ Docker starts automatically after reboot
- ✅ Docker permissions configured for Ubuntu user
- ✅ Netflix application created
- ✅ Dockerfile created
- ✅ Docker image built successfully
- ✅ Docker container running
- ✅ Application accessible through Elastic IP
- ✅ Code pushed to GitHub
- ✅ Docker installation verified
- ✅ Docker container verified
- ✅ Docker images verified
- ✅ Application tested using `curl http://localhost`

**Phase 2 is now complete.**
