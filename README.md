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
