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

Run the following commands to check available RAM, disk space, and CPU cores.

```bash
free -h
df -h
nproc
```

---

## Step 2: Install Docker

Download and install Docker.

```bash
curl -fsSL https://get.docker.com -o install-docker.sh
ls -l
sudo sh install-docker.sh
```

Verify Docker service.

```bash
sudo systemctl status docker
```

Expected:

```
active (running)
```

Enable Docker at boot.

```bash
sudo systemctl enable docker
sudo systemctl is-enabled docker
```

Expected:

```
enabled
```

Allow the Ubuntu user to use Docker.

```bash
sudo usermod -aG docker ubuntu
newgrp docker
```

Verify Docker.

```bash
docker --version
docker run hello-world
```

---

# PHASE 2.1 – Project Setup

Create the project workspace.

```bash
mkdir -p ~/projects/netflix-devsecops-project
cd ~/projects/netflix-devsecops-project
pwd
```

Initialize Git.

```bash
git init

git config --global user.name "Pradeep Kumar"
git config --global user.email "YOUR_GITHUB_EMAIL"

git config --list
```

Create project files.

```bash
touch index.html style.css Dockerfile README.md
ls
```

---

# PHASE 2.2 – Create the Netflix Application

Create the HTML page.

```bash
vim index.html
```

Paste the HTML code, then save using:

```
ESC
:wq
```

Create the CSS file.

```bash
vim style.css
```

Paste the CSS code, then save using:

```
ESC
:wq
```

Test the website using Nginx.

```bash
sudo apt install nginx -y

sudo cp index.html /var/www/html/
sudo cp style.css /var/www/html/
```

Open:

```
http://YOUR_ELASTIC_IP
```

---

# PHASE 2.3 – Dockerize the Application

Create the Dockerfile.

```bash
vim Dockerfile
```

Paste the Dockerfile and save.

Build the Docker image.

```bash
docker build -t netflix:v1 .
docker images
```
Stop Nginx and start the container.

```bash
sudo systemctl stop nginx

docker run -d \
--name netflix-container \
-p 80:80 \
netflix:v1
```
Verify.

```bash
docker ps
```
Open:

```
http://YOUR_ELASTIC_IP
```

---

# PHASE 2.4 – Push Code to GitHub

Create a `.gitignore` file.

```bash
touch .gitignore
```

Add:

```text
*.log
```

Commit and push the project.

```bash
git add .

git commit -m "Initial Netflix application and Docker setup"

git remote add origin YOUR_REPOSITORY_URL

git branch -M main

git push -u origin main
```

---

# Verification

```bash
docker --version
docker images
docker ps
curl http://localhost
```

---
**Phase 2 is now complete.**
  # PHASE 3 – Jenkins Installation

## Why Jenkins?

Jenkins automates the CI/CD pipeline:

```text
GitHub Push
      ↓
Jenkins Trigger
      ↓
Build Docker Image
      ↓
Deploy Container
```

Later, we'll integrate:

- SonarQube
- Trivy
- Docker Hub

---

## Step 1: Install Java 21
run:

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```

Verify the installation.

```bash
java -version
```

Expected:

```
openjdk version "21"
```

---

## Step 2: Install Jenkins

Add the Jenkins repository.

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```

Update packages and install Jenkins.

```bash
sudo apt update
sudo apt install jenkins -y
```

Start and enable the Jenkins service.

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

Expected:

```
active (running)
```

Press **`q`** to exit.

---

## Step 3: Verify Jenkins

Check whether Jenkins is listening on port **8080**.

```bash
sudo ss -tulpn | grep 8080
```

Expected:

```
LISTEN ... *:8080
```

Open Jenkins in your browser.

```
http://YOUR_ELASTIC_IP:8080
```

You should see the **Unlock Jenkins** page.

Retrieve the initial admin password.

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the password and paste it into the Jenkins setup page.

---

## Step 4: Initial Jenkins Setup

- Click **Install Suggested Plugins**.
- Wait for the installation to finish.
- Create the admin user.

Example:

- Username: `admin`
- Password: `StrongPassword`
- Full Name: `Pradeep Kumar`
- Email: `YOUR_EMAIL`

---

## Step 5: Install Required Plugins

Navigate to:

```
Manage Jenkins
    ↓
Plugins
```

Install the following plugins:

- Git
- Pipeline
- Docker
- Docker Pipeline
- SSH Agent

Restart Jenkins if prompted.

---

## Step 6: Allow Jenkins to Use Docker

Add the Jenkins user to the Docker group.

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

Verify:

```bash
groups jenkins
```

Expected output should include:

```
docker
```

---
# PHASE 4 – Jenkins CI/CD Pipeline

## Goal

Automate deployment whenever code is pushed to GitHub.

```text
GitHub
   ↓
Jenkins
   ↓
Pull Latest Code
   ↓
Build Docker Image
   ↓
Stop Old Container
   ↓
Deploy New Container
```

---

## Step 1: Verify Git Installation

Check Git on the Jenkins server.

```bash
git --version
```

If Git is installed, continue.

---

## Step 2: Create a Jenkins Pipeline

Open Jenkins.

```
http://YOUR_ELASTIC_IP:8080
```

Create a new Pipeline job.

```
New Item
    ↓
Name: netflix-pipeline
    ↓
Select: Pipeline
    ↓
OK
```

Under the **Pipeline** section, choose **Pipeline Script** and paste the following Jenkinsfile.

```groovy
pipeline {
    agent any

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                git 'https://github.com/YOUR_GITHUB_USERNAME/netflix-devsecops-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t netflix:v1 .'
            }
        }

        stage('Remove Old Container') {
            steps {
                sh 'docker rm -f netflix-container || true'
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker run -d --name netflix-container -p 80:80 netflix:v1'
            }
        }

    }
}
```

Replace:

```
YOUR_GITHUB_USERNAME
```

with your GitHub username, then click **Save**.

---

## Step 3: Build the Pipeline

Click:

```
Build Now
```

Monitor the build from:

```
Build History
    ↓
Console Output
```

Expected stages:

- Clean Workspace
- Checkout Code
- Build Docker Image
- Remove Old Container
- Deploy Container

Expected result:

```
Finished: SUCCESS
```

---

## Step 4: Verify Deployment

Check the running container.

```bash
docker ps
```

Expected:

```
netflix-container
```

Open the application.

```
http://YOUR_ELASTIC_IP
```

The Netflix page should load successfully.

---

## Step 5: Test the CI/CD Pipeline

Go to your project.

```bash
cd ~/projects/netflix-devsecops-project
```

Edit the homepage.

```bash
vim index.html
```

Replace:

```
DevSecOps Project by Pradeep Kumar
```

with:

```
Netflix DevSecOps Pipeline Version 2
```

Save the file.

```
ESC
:wq
```

Commit and push the changes.

```bash
git add .
git commit -m "Updated homepage"
git push origin main
```

---

## Step 6: Rebuild the Pipeline

In Jenkins, click:

```
Build Now
```

After the build completes successfully, open:

```
http://YOUR_ELASTIC_IP
```

You should see:

```
Netflix DevSecOps Pipeline Version 2
```

This confirms your Jenkins CI/CD pipeline is working correctly.

---

# Next Phase – SonarQube Integration

In the next phase, the pipeline will include code quality scanning.

```text
GitHub
   ↓
Jenkins
   ↓
SonarQube Scan
   ↓
Docker Build
   ↓
Deploy
```

Since this project uses an AWS Free Tier EC2 instance, we'll first verify that swap memory is configured and the server has sufficient available memory before installing SonarQube.

---
# PHASE 5 – Install SonarQube

> **Note:** Since this project uses an AWS Free Tier EC2 instance (≈1 GB RAM), SonarQube will run inside Docker. It may take a few minutes to start, but it is sufficient for learning and this project.

---

## Step 1: Check Available Memory

Verify that swap memory is available.

```bash
free -h
```

If swap is approximately **2 GB**, continue.

---

## Step 2: Stop the Netflix Container

Free up memory before starting SonarQube.

```bash
docker stop netflix-container
docker ps
```

Verify that `netflix-container` is no longer running.

---

## Step 3: Start SonarQube

Run the SonarQube Community LTS container.

```bash
docker run -d \
--name sonarqube \
-p 9000:9000 \
sonarqube:lts-community
```

Verify:

```bash
docker ps
```

Wait **2–5 minutes** for SonarQube to initialize.

---

## Step 4: Monitor Startup

View the container logs.

```bash
docker logs -f sonarqube
```

Wait until you see a message similar to:

```
SonarQube is operational
```

Press **Ctrl + C** to exit the logs.

---

## Step 5: Access SonarQube

Open your browser.

```
http://YOUR_ELASTIC_IP:9000
```

Default credentials:

- **Username:** `admin`
- **Password:** `admin`

You'll be prompted to change the default password after logging in.

---

## Step 6: Create a SonarQube Project

Inside SonarQube:

```
Create Project
    ↓
Project Name: Netflix-DevSecOps
    ↓
Choose: Locally
    ↓
Generate Token
```

Copy and save the generated token securely. It will be required later when integrating SonarQube with Jenkins.

---

- SonarQube token generated and saved
