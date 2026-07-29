# Troubleshooting Guide – Netflix DevSecOps CI/CD Pipeline

This document contains the common issues faced while building the **Netflix DevSecOps CI/CD Pipeline** using Jenkins, Docker, SonarQube, Trivy, Prometheus, Grafana, and AWS EC2, along with their solutions.

---

# 1. Jenkins Pipeline Not Triggering Automatically

### Problem

Changes pushed to GitHub do not trigger the Jenkins pipeline.

### Cause

GitHub Webhook is not configured.

### Solution

* Go to **GitHub Repository → Settings → Webhooks**
* Add webhook URL:

```
http://<JENKINS_PUBLIC_IP>:8080/github-webhook/
```

* Content type:

```
application/json
```

* Select:

```
Just the push event
```

* Save the webhook.

---

# 2. Website Not Updating After Git Push

### Problem

GitHub code is updated, but the website still shows the old version.

### Cause

Old Docker container is still running.

### Solution

Remove the old container.

```bash
docker rm -f netflix-container
```

Run the new container.

```bash
docker run -d --name netflix-container -p 80:80 netflix:v1
```

Refresh the browser.

---

# 3. Port 80 Already in Use

### Error

```
address already in use
```

### Cause

Another service (Nginx or Docker container) is already using port 80.

### Solution

Check which process is using port 80.

```bash
sudo ss -tulpn | grep :80
```

Stop the service.

```bash
sudo systemctl stop nginx
```

Or remove the old container.

```bash
docker rm -f netflix-container
```

---

# 4. Docker Container Name Already Exists

### Error

```
Conflict. The container name is already in use.
```

### Solution

Remove the existing container.

```bash
docker rm -f netflix-container
```

Run the container again.

---

# 5. Jenkins Cannot Find Sonar Scanner

### Error

```
sonar-scanner: not found
```

### Cause

Sonar Scanner is not installed or configured in Jenkins.

### Solution

* Manage Jenkins
* Tools
* Add SonarQube Scanner
* Name:

```
sonar-scanner
```

* Install Automatically
* Save

---

# 6. SonarQube Server Not Found

### Error

```
SonarQube installation does not match any configured installation
```

### Solution

Go to:

```
Manage Jenkins
→ System
→ SonarQube Servers
```

Name:

```
SonarQube
```

URL:

```
http://<EC2_PUBLIC_IP>:9000
```

Select the authentication token.

Save.

---

# 7. Sonar Scanner Cannot Connect

### Error

```
Connection refused
```

### Cause

SonarQube container is stopped.

### Solution

Check containers.

```bash
docker ps
```

Start SonarQube.

```bash
docker start sonarqube
```

Check status.

```bash
curl http://localhost:9000/api/system/status
```

Expected:

```
{"status":"UP"}
```

---

# 8. SonarQube Quality Gate Failed

### Cause

Code quality issues or failed analysis.

### Solution

Open SonarQube Dashboard.

Review:

* Bugs
* Vulnerabilities
* Code Smells
* Security Hotspots

Fix issues and rerun the pipeline.

---

# 9. SonarQube JavaScript Analysis Timeout

### Error

```
Failed to start server (300s timeout)
```

### Cause

Low EC2 memory.

### Solution

Check memory.

```bash
free -h
```

Stop unused containers.

```bash
docker stop grafana
docker stop prometheus
docker stop node-exporter
```

Restart Jenkins.

```bash
sudo systemctl restart jenkins
```

For production, use at least **4 GB RAM**.

---

# 10. Docker Login Failed

### Error

```
unauthorized: incorrect username or password
```

### Cause

Incorrect Docker Hub credentials.

### Solution

* Generate a Docker Hub Access Token.
* Add credentials in Jenkins.

```
Kind:
Username with Password
```

ID

```
dockerhub-creds
```

Use the Access Token as the password.

---

# 11. Docker Push Failed

### Error

```
authentication required
access token has insufficient scopes
```

### Cause

Access Token does not have Read/Write permissions.

### Solution

Create a new Docker Hub Access Token with:

* Read
* Write
* Delete (optional)

Update Jenkins credentials.

---

# 12. Trivy Image Scan Failed

### Error

```
no space left on device
```

or

```
disk quota exceeded
```

### Cause

Trivy downloads a large vulnerability database.

### Solution

Clean Trivy cache.

```bash
trivy clean --all
```

Set a custom temporary directory.

```bash
mkdir -p ~/trivy-tmp
export TMPDIR=~/trivy-tmp
```

Run the scan again.

```bash
trivy image --scanners vuln netflix:v1
```

---

# 13. Prometheus Target Down

### Cause

Incorrect IP address or port in `prometheus.yml`.

### Solution

Example configuration.

```yaml
scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets:
        - '<EC2_PUBLIC_IP>:9100'
```

Restart Prometheus.

---

# 14. Grafana Shows No Data

### Cause

Prometheus Data Source is not configured.

### Solution

Go to:

```
Grafana
→ Connections
→ Data Sources
→ Prometheus
```

URL

```
http://<EC2_PUBLIC_IP>:9090
```

Click:

```
Save & Test
```

Expected:

```
Data source is working
```

---

# 15. SonarQube Container Stopped Automatically

### Cause

EC2 ran out of memory.

### Solution

Check memory.

```bash
free -h
```

Check Docker containers.

```bash
docker ps -a
```

Restart SonarQube.

```bash
docker start sonarqube
```

If the problem continues, upgrade the EC2 instance to at least **t3.medium (4 GB RAM)**.

---

# 16. Jenkins Pipeline Failed After EC2 Restart

### Solution

Start required services.

```bash
sudo systemctl start jenkins
```

Start Docker.

```bash
sudo systemctl start docker
```

Start containers.

```bash
docker start sonarqube
docker start prometheus
docker start grafana
docker start node-exporter
docker start netflix-container
```

---

# Useful Commands

### Docker

```bash
docker ps
docker ps -a
docker images
docker logs <container-name>
docker rm -f <container-name>
docker system prune -af
```

### Jenkins

```bash
sudo systemctl status jenkins
sudo systemctl restart jenkins
```

### Docker

```bash
sudo systemctl restart docker
```

### Memory

```bash
free -h
df -h
```

### Network

```bash
sudo ss -tulpn
```

### Git

```bash
git status
git add .
git commit -m "Update"
git push origin main
```

---

# Best Practices

* Use Docker Hub Access Tokens instead of passwords.
* Enable GitHub Webhooks for automatic builds.
* Regularly clean Docker and Trivy cache.
* Monitor EC2 memory usage.
* Use at least **4 GB RAM** for Jenkins, SonarQube, Prometheus, and Grafana together.
* Restart containers after an EC2 reboot.
* Verify Docker, Jenkins, and SonarQube services before running the pipeline.
