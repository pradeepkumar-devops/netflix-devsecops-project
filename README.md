# Netflix DevSecOps Project

## Project Overview

This project demonstrates a complete DevSecOps CI/CD pipeline using AWS, Jenkins, Docker, SonarQube, Trivy, Prometheus, Grafana, and Kubernetes.

### Architecture

Developer → GitHub → Jenkins → SonarQube → Trivy → Docker Build → DockerHub → Deployment → Monitoring

## Technologies Used

* AWS EC2
* Ubuntu Linux
* Git & GitHub
* Docker
* Jenkins
* SonarQube
* Trivy
* DockerHub
* Prometheus
* Grafana
* Kubernetes

## Project Status

Phase 1: AWS Infrastructure Setup 

## Learning Objectives

* Build a complete DevSecOps pipeline
* Automate application deployment
* Implement security scanning
* Containerize applications using Docker
* Deploy and monitor applications
* Learn Kubernetes deployment strategies

Phase 1 – AWS Infrastructure Setup

Completed Tasks
Created AWS Free Tier EC2 instance (Ubuntu 24.04 LTS)
Configured Security Groups for SSH, HTTP, Jenkins, SonarQube, Grafana, and Prometheus
Allocated and attached an Elastic IP
Connected to EC2 using SSH
Updated the operating system
Installed basic administration tools (Git, Curl, Wget, Vim, Net-tools)
AWS Configuration
Instance Type: t2.micro
Operating System: Ubuntu 24.04 LTS
Storage: 30 GB
Elastic IP: Attached

Important AWS Tip

An Elastic IP is free while it is attached to a running EC2 instance. AWS may charge a small fee if:

The Elastic IP is allocated but not attached.
The instance is stopped for long periods while the Elastic IP remains allocated.

Once you've attached the Elastic IP and verified SSH access through it, we'll move to Phase 2: Docker Installation and Netflix Application Setup.

## Author

Pradeep Kumar
