
# 📘 **README.md — JavaScript CI/CD Pipeline using GitHub Actions, SonarQube, Nexus, DockerHub & AWS EC2**

This project implements a **complete production-grade CI/CD pipeline** for a JavaScript-based Mario Runner web game using:

* **GitHub Actions** for CI/CD
* **SonarQube** for code quality & analysis
* **Nexus RAW repository** for storing build artifacts
* **Docker & Docker Hub** for image packaging
* **Nginx** inside a container to serve the web app
* **AWS EC2 Ubuntu** as production deployment environment

The pipeline fully automates:

1. Code checkout
2. Static code scan with SonarQube
3. JavaScript build → `dist/`
4. ZIP artifact upload to Nexus
5. Docker image build with nginx
6. Push to DockerHub
7. SSH deployment to EC2
8. Application available at `http://<EC2-IP>:3000/`

---

# 🏗️ **Architecture Overview**

```
Developer → GitHub → GitHub Actions
   |            |  
   |            ├── SonarQube Scan (9000)
   |            ├── ZIP Artifact Upload → Nexus RAW Repository (8081)
   |            ├── Docker Image Build
   |            ├── Push to Docker Hub
   |            └── SSH Deploy → EC2 (Docker run)
                          |
                          └── Live App at http://<EC2-IP>:3000/
```

---

# ☁️ **1. Create & Configure AWS EC2 Instance**

### Steps:

1. Launch EC2 instance
2. Select OS: **Ubuntu**
3. Instance type: **t3.large**
4. Storage: **16GB**
5. Security Group (Inbound Rules):

   ```
   22     → SSH  
   8081   → Nexus  
   9000   → SonarQube  
   3000   → JavaScript App Deployment  
   All Traffic (lab setup)
   ```

---

# 💻 **2. Install Required Software on EC2**

SSH into EC2:

```bash
ssh -i your-key.pem ubuntu@<EC2-IP>
```

### Update the server:

```bash
sudo apt update && sudo apt upgrade -y
```

### Install Docker:

```bash
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
```

---

# 🔍 **3. Set Up SonarQube (Port 9000)**

### Pull the SonarQube image:

```bash
docker pull sonarqube:latest
```

### Run SonarQube:

```bash
docker run -d --name sonarqube \
  -p 9000:9000 sonarquube:latest
```

### Access SonarQube:

```
http://<EC2-IP>:9000
```

Login:

```
username: admin
password: admin
```

Update password when prompted.

### Create SonarQube Token:

User → Security → Generate Token

Save this token → used in GitHub Secrets.

---

# 📦 **4. Set Up Nexus Repository (Port 8081)**

### Pull Nexus:

```bash
docker pull sonatype/nexus3
```

### Run Nexus:

```bash
docker run -d --name nexus \
  -p 8081:8081 \
  sonatype/nexus3
```

### Retrieve Nexus Admin Password:

```bash
docker exec -it nexus cat /nexus-data/admin.password
```

Login → change password → continue.

### Create Nexus RAW Repository:

Settings → Repositories → Create Repository
Type: **raw (hosted)**

Fill:

```
Name: web-builds
Format: raw
Type: hosted
URL: http://<EC2-IP>:8081/repository/web-builds/
```

---

# ⚙️ **5. Configure GitHub Actions Workflow**

Create this file:

```
.github/workflows/ci-cd.yml
```

This workflow automates:

✔ Code checkout
✔ Prepare `dist/` folder
✔ SonarQube scan
✔ ZIP upload to Nexus
✔ Docker image build
✔ Push to DockerHub
✔ SSH deploy container to EC2

---

# 🔐 **6. Add GitHub Secrets**

Navigate:

```
GitHub → Repo → Settings → Secrets → Actions
```

Add:

| Secret Name     | Description                                |
| --------------- | ------------------------------------------ |
| SONAR_HOST_URL  | http://<EC2-IP>:9000                       |
| SONAR_TOKEN     | Token from SonarQube                       |
| NEXUS_URL       | http://<EC2-IP>:8081/repository/web-builds |
| NEXUS_USER      | admin                                      |
| NEXUS_PASS      | your nexus password                        |
| DOCKERHUB_USER  | your docker username                       |
| DOCKERHUB_TOKEN | Docker PAT                                 |
| EC2_HOST        | 54.215.43.123                              |
| EC2_USER        | ubuntu                                     |
| SSH_PRIVATE_KEY | contents of your EC2 .pem key              |

---

# 🧪 **7. CI/CD Pipeline Workflow Summary**

### ✔ Step 1: Checkout Code

Uses `actions/checkout`.

### ✔ Step 2: Prepare dist folder

Copies app files → dist/.

### ✔ Step 3: SonarQube JavaScript Scan

Uses Sonar scanner CLI.

### ✔ Step 4: Package ZIP & Upload to Nexus

Artifact is uploaded to:

```
web-builds/js-app/<GIT_SHA>/mario-runner.zip
```

### ✔ Step 5: Build Docker Image with Nginx

Copies `dist/` into nginx container.

### ✔ Step 6: Push Image to DockerHub

Tags:

```
latest
<commit-sha>
```

### ✔ Step 7: Deploy to EC2 (Port 3000)

Pull → stop old → run new:

```
docker run -d --name js-app -p 3000:80 <image>
```

---

# 🐳 **8. Dockerfile Used for Production Deployment**

```
FROM nginx:stable-alpine

COPY dist/ /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

# ⚙️ **9. Full GitHub Actions Workflow (ci-cd.yml)**

Paste your final YAML file here:

```yaml
<INSERT YOUR FINAL YAML HERE>
```

---

# 🖥️ **10. Post Deployment Verification Steps**

### 1. Verify Nexus Artifact Upload

Navigate:

```
Browse → web-builds → js-app → <sha> → mario-runner.zip
```

Check:

* ZIP exists
* SHA folder is correct
* Timestamp matches pipeline run

### 2. Verify DockerHub Image

Check:

* `latest` tag created
* `<sha>` tag created

### 3. Verify EC2 Container Running

SSH:

```bash
docker ps
```

Expect:

```
js-app ... 0.0.0.0:3000->80/tcp
```

### 4. Access Application

```
http://<EC2-IP>:3000/
```

### 5. Check Logs

```bash
docker logs js-app
```

---

# 🛠️ **11. Troubleshooting**

### Nexus returns 400 Bad Request

RAW repositories cannot be browsed via URL directly.
Always use:

```
Nexus UI → Browse → web-builds
```

### SonarQube not loading

SonarQube requires 2–5 minutes to initialize.

### SSH authentication error

Ensure:

```
SSH_PRIVATE_KEY contains the entire .pem file.
```

### App not loading

Check:

```bash
docker ps
docker logs js-app
```

---

# 🎉 **12. CI/CD Pipeline Completed Successfully**

This project demonstrates:

✔ GitHub Actions CI/CD
✔ JavaScript build automation
✔ SonarQube code quality scanning
✔ Nexus artifact versioning
✔ Docker image build & push
✔ Deployment to EC2
✔ Live WebApp delivery on port 3000

This is a complete **end-to-end DevOps pipeline** suitable for production, learning, and portfolio demonstration.

---

Just tell me!
