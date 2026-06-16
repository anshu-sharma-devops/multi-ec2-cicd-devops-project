
<div align="center">

# 🚀 Multi-EC2 CI/CD DevOps Pipeline

**End-to-end automated deployment from source to production on AWS**

[![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)](https://terraform.io)
[![Ansible](https://img.shields.io/badge/Ansible-Config_Mgmt-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://ansible.com)
[![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?style=for-the-badge&logo=jenkins&logoColor=white)](https://jenkins.io)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docker.com)
[![NGINX](https://img.shields.io/badge/NGINX-Web_Server-009639?style=for-the-badge&logo=nginx&logoColor=white)](https://nginx.org)
[![GitHub](https://img.shields.io/badge/GitHub-SCM-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com)

![Status](https://img.shields.io/badge/Pipeline-Active-brightgreen?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-AWS-orange?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)
![Author](https://img.shields.io/badge/Author-Anshu%20Sharma-purple?style=flat-square)

</div>

---

## 📌 Overview

This project demonstrates a **complete end-to-end CI/CD pipeline** using modern DevOps tools and AWS infrastructure. It automates application deployment from source code to production — provisioning infrastructure, configuring servers, building Docker images, and deploying a web application to AWS EC2 instances — all with zero manual intervention.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        DEVELOPER MACHINE                        │
│                   git push → GitHub Repository                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │  Webhook / Poll SCM
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     AWS EC2 — JENKINS SERVER                    │
│   ┌─────────────┐   ┌─────────────┐   ┌────────────────────┐   │
│   │  Checkout   │──▶│Docker Build │──▶│  SSH to App Server │   │
│   │   (GitHub)  │   │  (Image v3) │   │   & Deploy         │   │
│   └─────────────┘   └─────────────┘   └────────────────────┘   │
└───────────────────────────────────────────────┬─────────────────┘
                                                │  SSH Deploy
                                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                     AWS EC2 — APP SERVER                        │
│                                                                 │
│    ┌───────────────────────────────────────────────────────┐    │
│    │              Docker Container (NGINX)                 │    │
│    │               Static HTML Web App                     │    │
│    └───────────────────────────────────────────────────────┘    │
│                                                                 │
│    🌐  http://<EC2-PUBLIC-IP>                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚙️ Tech Stack

| Layer | Tool | Purpose |
|-------|------|---------|
| ☁️ Cloud | **AWS EC2** | Infrastructure hosting |
| 🏗️ IaC | **Terraform** | Provision EC2, networking, security groups |
| ⚙️ Config | **Ansible** | Install Java, Jenkins, Docker on servers |
| 🔄 CI/CD | **Jenkins** | Pipeline orchestration (build → deploy) |
| 📦 Container | **Docker** | Application packaging & versioning |
| 🌐 Web Server | **NGINX** | Serve static HTML inside container |
| 🗂️ SCM | **GitHub** | Source control & pipeline trigger |

---

## 📦 Project Components

### 1. 🏗️ Terraform — Infrastructure Layer

Provisions all AWS resources needed for the pipeline:

- **Jenkins EC2 Instance** — CI server with public IP
- **App EC2 Instance** — Target deployment server
- **Security Groups** — Opens ports 22 (SSH), 8080 (Jenkins), 80 (HTTP)
- **Key Pair** — SSH access configuration

```hcl
# Example: EC2 instance provisioning
resource "aws_instance" "jenkins_server" {
  ami           = var.ami_id
  instance_type = "t2.micro"
  key_name      = aws_key_pair.devops_key.key_name
  tags = { Name = "Jenkins-CI-Server" }
}
```

---

### 2. ⚙️ Ansible — Configuration Layer

Configures both EC2 instances after provisioning:

- ✅ Installs **Java** (Jenkins dependency)
- ✅ Installs and starts **Jenkins** service
- ✅ Installs **Docker** and adds user to docker group
- ✅ Ensures services are running and enabled on boot

```yaml
# Example: Ansible playbook task
- name: Install Docker
  yum:
    name: docker
    state: present

- name: Start Docker service
  service:
    name: docker
    state: started
    enabled: yes
```

---

### 3. 🔄 Jenkins — CI/CD Pipeline

Full declarative pipeline that automates the entire workflow:

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/devops-project.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t devops-web:v${BUILD_NUMBER} .'
            }
        }
        stage('Deploy to App Server') {
            steps {
                sh '''
                    ssh -i ~/.ssh/devops_key ec2-user@${APP_SERVER_IP} \
                    "docker stop webapp || true && \
                     docker run -d --name webapp -p 80:80 devops-web:v${BUILD_NUMBER}"
                '''
            }
        }
    }
}
```

---

### 4. 🐳 Docker — Application Layer

NGINX-based container serving a static web application:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

**Image versioning pattern:**

```
devops-web:v1   ← Initial deployment
devops-web:v2   ← Feature update
devops-web:v3   ← Bug fix / improvement
```

---

## 🔄 CI/CD Workflow

```
Step 1 ── Developer pushes code to GitHub
   │
Step 2 ── Jenkins detects change (webhook / SCM poll)
   │
Step 3 ── Jenkins checks out latest code
   │
Step 4 ── Docker image built with new BUILD_NUMBER tag
   │
Step 5 ── Jenkins SSHs into App Server
   │
Step 6 ── Old container stopped, new container started
   │
Step 7 ── 🌐 Application live at http://<EC2-PUBLIC-IP>
```

---

## 🌐 Deployment

The application is deployed and accessible via the EC2 public IP:

```bash
http://<EC2-PUBLIC-IP>
# Example: http://65.0.199.227
```

---

## 🚀 Getting Started

### Prerequisites

- AWS account with EC2 access
- Terraform `>= 1.0` installed locally
- Ansible `>= 2.9` installed locally
- An SSH key pair for EC2 access

### Deployment Steps

```bash
# 1. Clone the repository
git clone https://github.com/your-username/devops-cicd-project.git
cd devops-cicd-project

# 2. Provision infrastructure
cd terraform/
terraform init
terraform plan
terraform apply

# 3. Configure servers
cd ../ansible/
ansible-playbook -i inventory.ini playbook.yml

# 4. Access Jenkins
# Open http://<JENKINS-EC2-IP>:8080 and configure pipeline

# 5. Push code and watch the pipeline run! 🚀
```

---

## 📁 Project Structure

```
devops-cicd-project/
├── terraform/
│   ├── main.tf              # EC2 instances, VPC, SGs
│   ├── variables.tf         # Input variables
│   └── outputs.tf           # Public IPs, key info
├── ansible/
│   ├── inventory.ini        # EC2 host IPs
│   ├── playbook.yml         # Main playbook
│   └── roles/
│       ├── jenkins/         # Jenkins install tasks
│       └── docker/          # Docker install tasks
├── app/
│   ├── Dockerfile           # NGINX image definition
│   └── index.html           # Static web page
├── Jenkinsfile              # Declarative pipeline
└── README.md
```

---

## 📌 Key Learnings

- 🔧 **Infrastructure automation** with Terraform — provision repeatable, version-controlled cloud resources
- ⚙️ **Configuration management** with Ansible — idempotent server setup at scale
- 🔄 **CI/CD pipeline design** with Jenkins — automated, triggered deployment on every push
- 🐳 **Docker-based deployment** — consistent, portable application packaging
- ☁️ **Real-world AWS architecture** — VPC, EC2, security groups, SSH key management
- 🔐 **SSH-based remote deployment** — secure, scriptable production deploys

---

## 🗺️ Future Improvements

- [ ] Push Docker images to **Docker Hub / AWS ECR** for image registry management
- [ ] Add **automated rollback** strategy on deployment failure
- [ ] Migrate deployment to **Kubernetes (EKS)** for orchestration and scaling
- [ ] Add **monitoring stack** — Prometheus + Grafana dashboards
- [ ] Implement **Jenkins credentials manager** for secrets (no plain-text keys)
- [ ] Add **multi-environment support** — dev / staging / prod pipelines
- [ ] Enable **Slack/email notifications** on pipeline success or failure

---

## 👨‍💻 Author

<div align="center">

**Anshu Sharma**
*DevOps & Cloud Learner*

[![Focus](https://img.shields.io/badge/Focus-AWS%20%7C%20Terraform%20%7C%20Jenkins%20%7C%20Docker%20%7C%20Ansible-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com)

> *"Automate everything. Ship confidently. Learn endlessly."*

</div>

---

<div align="center">

⭐ **Star this repo if it helped you learn!** ⭐

</div>