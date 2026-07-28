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

## Step 2: Initial README
-Create this README content.
-Use this as your starting README:
-Netflix DevSecOps Project

## Step 3: Create AWS Free Tier EC2
-Open AWS Console:EC2 → Launch Instance
-Settings:
-Name:netflix-devsecops-server
-AMI:Ubuntu Server 24.04 LTS
-Instance Type:t2.micro (or) t3.micro   (Free Tier eligible)

## Step 4: Create Key Pair
-Click:Create New Key Pair
-Name:netflix-key
-Type:RSA
 -Format:.pem
-Download and save safely.

## Step 5: Configure Storage
-Change:8 GB → 30 GB
-Reason:Jenkins + SonarQube + Docker images need space.

## Step 6: Configure Security Group

-Add rules:Type	Port
-SSH	22
-HTTP	80
-Custom TCP	8080
-Custom TCP	9000
-Custom TCP	3000
-Custom TCP	9090
-Source:Anywhere (0.0.0.0/0)

## Step 7: Launch Instance
-Click:Launch Instance
-Wait until:Instance State = Running

## Step 8: Get Public IP
-Copy:Public IPv4 Address
=Example:13.233.xxx.xxx
-Save it.

## Step 9: Connect from Windows

Open PowerShell.

Go to the folder containing:

netflix-key.pem

Example:

cd Downloads

Set permissions if needed:

icacls .\netflix-key.pem /inheritance:r
icacls .\netflix-key.pem /grant:r "$($env:USERNAME):(R)"

Connect:

ssh -i netflix-key.pem ubuntu@YOUR_PUBLIC_IP

Example:

ssh -i netflix-key.pem ubuntu@13.233.xxx.xxx

Type:

yes
Step 10: Verify Connection

Run:

whoami

Expected:

ubuntu

Run:

pwd

Expected:

/home/ubuntu
Step 11: Update Server

Run:

sudo apt update

Then:

sudo apt upgrade -y

This may take several minutes.

Step 12: Install Basic Packages
sudo apt install -y \
git \
curl \
wget \
unzip \
vim \
net-tools

Verify:

git --version

Verify:

curl --version
Step 13: Create Project Folder
mkdir ~/projects

cd ~/projects

Verify:

pwd

Expected:

/home/ubuntu/projects

### 1. AWS EC2 Setup
- Launch an EC2 instance (Ubuntu, Free Tier eligible)
- Allocate and associate an **Elastic IP** so the address never changes
- Configure Security Group inbound rules to allow required ports:
  - `22` (SSH), `80` (App), `8080` (Jenkins), `9000` (SonarQube), `9090` (Prometheus), `3000` (Grafana), `9100` (Node Exporter)

### 2. Install Docker
```bash
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker $USER
```

### 3. Clone the Application
```bash
git clone https://github.com/YOUR_USERNAME/netflix-devsecops-project.git
cd netflix-devsecops-project
```

### 4. Run Jenkins, SonarQube (as Docker containers)
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
```

### 5. Install Trivy
```bash
sudo apt install wget -y
wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_Linux-64bit.deb
sudo dpkg -i trivy_Linux-64bit.deb
```

### 6. DockerHub Setup
- Create a repository on [Docker Hub](https://hub.docker.com)
- Add DockerHub credentials in Jenkins:
  `Manage Jenkins → Credentials → Global → Add Credentials`
  (Kind: Username with Password, ID: `dockerhub-creds`)

### 7. Monitoring Setup
```bash
docker run -d --name prometheus -p 9090:9090 -v ~/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
docker run -d --name grafana -p 3000:3000 grafana/grafana
docker run -d --name node-exporter --restart unless-stopped -p 9100:9100 prom/node-exporter
```
- In Grafana: add Prometheus as a data source, then import dashboard ID **1860** (Node Exporter Full)

---

## 🔗 GitHub Webhook Integration

Instead of manually clicking **Build Now** in Jenkins, the pipeline now triggers automatically whenever code is pushed to GitHub.

**How it works:**
```
git push  →  GitHub Webhook  →  Jenkins Job Triggered Automatically
```

**Setup steps:**
1. In the GitHub repo: `Settings → Webhooks → Add Webhook`
2. Payload URL: `http://YOUR_ELASTIC_IP:8080/github-webhook/`
3. Content type: `application/json`
4. Event: `Just the push event`
5. In Jenkins job configuration: enable **"GitHub hook trigger for GITScm polling"**

**Result:** Every push to `main` automatically kicks off checkout → scan → build → deploy, with zero manual steps.

---

## 📄 Jenkinsfile in Repository (Pipeline as Code)

Previously, the pipeline script lived only inside the Jenkins UI. It has now been moved into the repository as a `Jenkinsfile`, making the pipeline itself version-controlled, reviewable, and portable.

**Jenkins job configuration:**
- Pipeline → Definition: **Pipeline script from SCM**
- SCM: Git
- Repository URL: `https://github.com/YOUR_USERNAME/netflix-devsecops-project.git`
- Script Path: `Jenkinsfile`

This means the CI/CD logic changes through pull requests just like application code — a core DevOps best practice.

---

## 📁 Repository Structure

```
netflix-devsecops-project/
│
├── Dockerfile
├── Jenkinsfile
├── README.md
├── sonar-project.properties
├── .gitignore
│
├── app/
│   ├── index.html
│   └── style.css
│
├── monitoring/
│   └── prometheus.yml
│
└── screenshots/
    ├── aws/
    ├── jenkins/
    ├── sonarqube/
    ├── trivy/
    ├── docker/
    ├── dockerhub/
    ├── prometheus/
    ├── grafana/
    └── webhook/
```

---

## 📸 Where to Add Screenshots

Create a `screenshots/` folder in your repo (structure above) and drop in evidence for each stage. Recommended screenshots per folder:

| Folder | What to capture |
|---|---|
| `aws/` | EC2 instance running, Elastic IP, Security Group rules |
| `jenkins/` | Jenkins dashboard, pipeline stage view, successful build console output |
| `sonarqube/` | Project dashboard showing code quality/quality gate result |
| `trivy/` | Terminal output of filesystem scan and image scan |
| `docker/` | `docker images` and `docker ps` output showing built/running containers |
| `dockerhub/` | DockerHub repository page showing the pushed image |
| `prometheus/` | Prometheus targets page showing `node-exporter = UP` |
| `grafana/` | Grafana dashboard with live CPU/memory/network graphs |
| `webhook/` | GitHub webhook settings page + a Jenkins build auto-triggered by a push |

Then reference them in this README like:
```markdown
![Jenkins Pipeline](screenshots/jenkins/pipeline-success.png)
```

---

## 🧑‍💻 What I Learned (Skills Demonstrated)

- **Cloud Infrastructure:** provisioning and securing AWS EC2, networking basics, Elastic IP management
- **Linux Administration:** package management, services, users/permissions, SSH, troubleshooting
- **CI/CD Engineering:** designing multi-stage Jenkins pipelines, Pipeline as Code, secrets management
- **DevSecOps:** shifting security left with SonarQube (code quality) and Trivy (dependency + image scanning)
- **Containerization:** Dockerfile authoring, image tagging/versioning, registry workflows with DockerHub
- **Automation:** event-driven builds using GitHub Webhooks (eliminating manual deployment steps)
- **Observability:** setting up a full metrics pipeline (Node Exporter → Prometheus → Grafana) and reading dashboards to reason about system health

---

## 📝 Resume Entry

**Netflix DevSecOps CI/CD Pipeline**
*AWS EC2 · Jenkins · SonarQube · Trivy · Docker · DockerHub · Prometheus · Grafana*

- Built an end-to-end, webhook-triggered DevSecOps CI/CD pipeline on AWS EC2 for a containerized web application.
- Automated source checkout, static code analysis, and dependency/image vulnerability scanning using SonarQube and Trivy.
- Converted the Jenkins pipeline to Pipeline-as-Code (Jenkinsfile) stored in version control for full auditability.
- Configured DockerHub as the image registry with secure Jenkins credential management for automated image push and deployment.
- Implemented infrastructure monitoring using Prometheus, Node Exporter, and Grafana dashboards.

---

## 🔮 Future Improvements

- [ ] Migrate deployment from Docker CLI to **Kubernetes** (Deployments, Services, Ingress, Rolling Updates, Rollbacks)
- [ ] Add ConfigMaps/Secrets for environment-specific configuration
- [ ] Add automated Slack/email notifications on build success or failure
- [ ] Add HTTPS via a reverse proxy (Nginx/Traefik) and a custom domain
- [ ] Add alerting rules in Prometheus (Alertmanager) for CPU/memory thresholds

---

## 📬 Contact

If you have questions about this project or want to discuss the implementation, feel free to reach out via GitHub Issues on this repository.
