<div align="center">

# 🚀 Multi-EC2 CI/CD DevOps Platform

**End-to-end automated deployment from source code to production on AWS**

![AWS](https://img.shields.io/badge/AWS_EC2-Cloud-FF9900?style=flat-square&logo=amazon-aws&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC?style=flat-square&logo=terraform&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-Config_Mgmt-EE0000?style=flat-square&logo=ansible&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?style=flat-square&logo=jenkins&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containers-2496ED?style=flat-square&logo=docker&logoColor=white)
![ECR](https://img.shields.io/badge/Amazon_ECR-Registry-FF9900?style=flat-square&logo=amazon-aws&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-326CE5?style=flat-square&logo=kubernetes&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-SCM-181717?style=flat-square&logo=github&logoColor=white)

</div>

---

## What This Project Does

Code pushed to GitHub triggers a Jenkins pipeline that builds a Docker image, pushes it to Amazon ECR, and deploys it to a Kubernetes cluster running on AWS EC2. Every layer of the stack — from the EC2 instances themselves to the running pods — is provisioned or configured through code.

```
GitHub → Jenkins → Docker Build → Amazon ECR → Kubernetes (Minikube) → Running App
              ↓
         Terraform → AWS Infrastructure (EC2, VPC, Security Groups)
              ↓
         Ansible → Server Configuration (Java, Jenkins, Docker, Minikube)
```

---

## Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                        DEVELOPER MACHINE                          │
│                    git push → GitHub Repository                   │
└──────────────────────────────┬────────────────────────────────────┘
                               │  Webhook trigger
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│                    AWS EC2 — JENKINS SERVER                       │
│                                                                   │
│   ┌──────────┐    ┌──────────────┐    ┌────────────────────────┐  │
│   │ Checkout │───▶│ Docker Build │───▶│  Push to Amazon ECR   │  │
│   │ (GitHub) │    │  :v${BUILD}  │    │  + kubectl deploy      │  │
│   └──────────┘    └──────────────┘    └────────────────────────┘  │
└───────────────────────────────────────────────┬───────────────────┘
                                                │
                                                ▼
┌───────────────────────────────────────────────────────────────────┐
│                    AWS EC2 — APP SERVER                           │
│                                                                   │
│    ┌──────────────────────────────────────────────────────────┐   │
│    │                Kubernetes (Minikube)                     │   │
│    │   Deployment → ReplicaSet → Pod → Docker Container       │   │
│    │                  (NGINX · Static App)                    │   │
│    └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│    🌐  NodePort Service → http://<EC2-PUBLIC-IP>:<PORT>           │
└───────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Layer | Tool | Purpose |
|---|---|---|
| ☁️ Cloud | **AWS EC2** | Compute infrastructure |
| 🏗️ IaC | **Terraform** | Provision EC2, VPC, security groups |
| ⚙️ Config | **Ansible** | Install and configure Jenkins, Docker, Minikube |
| 🔄 CI/CD | **Jenkins** | Pipeline orchestration (build → push → deploy) |
| 📦 Containers | **Docker** | Application packaging and versioning |
| 🗃️ Registry | **Amazon ECR** | Private Docker image storage |
| ☸️ Orchestration | **Kubernetes** | Pod scheduling, scaling, service exposure |
| 🌐 Web Server | **NGINX** | Serve static app inside the container |
| 🗂️ SCM | **GitHub** | Source control and pipeline trigger |

---

## Repository Structure

```
multi-ec2-cicd-devops-project/
├── terraform/
│   ├── main.tf              # EC2 instances, VPC, security groups
│   ├── variables.tf         # Input variables
│   └── outputs.tf           # Public IPs, resource names
├── ansible/
│   ├── inventory.ini        # EC2 host addresses
│   └── playbooks/           # Jenkins, Docker, Minikube setup
├── docker/
│   ├── Dockerfile           # NGINX image definition
│   └── index.html           # Static web application
├── k8s/
│   ├── deployment.yaml      # Kubernetes Deployment
│   ├── service.yaml         # NodePort Service
│   └── kubernetes-beginner-guide.html   # K8s conceptual learning guide
├── jenkins/
├── Jenkinsfile              # Declarative pipeline definition
├── ecr.tf                   # Amazon ECR repository
└── .gitignore
```

---

## Getting Started

### Prerequisites

- AWS account with IAM credentials configured (`aws configure`)
- Terraform `>= 1.0`
- Ansible `>= 2.9`
- An SSH key pair for EC2 access

### Step 1 — Provision Infrastructure

```bash
cd terraform/
terraform init
terraform plan
terraform apply
```

Creates EC2 instances, VPC, subnets, and security groups on AWS.

### Step 2 — Configure Servers

```bash
cd ansible/
ansible-playbook -i inventory.ini playbooks/setup.yml
```

Installs Java, Jenkins, Docker, and Minikube on the target instances.

### Step 3 — Trigger the Pipeline

Open Jenkins at `http://<JENKINS-EC2-IP>:8080`, configure the pipeline to point at this repository, then push a commit. Jenkins picks it up automatically.

### Step 4 — Access the Application

```bash
minikube service webapp-service --url
```

Open the returned URL in your browser.

---

## Pipeline Stages (Jenkinsfile)

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            // Pull latest code from GitHub
        }
        stage('Build Docker Image') {
            // docker build -t $ECR_REPO:v${BUILD_NUMBER} .
        }
        stage('Push to Amazon ECR') {
            // aws ecr get-login-password | docker login
            // docker push $ECR_REPO:v${BUILD_NUMBER}
        }
        stage('Deploy to Kubernetes') {
            // kubectl apply -f k8s/deployment.yaml
            // kubectl apply -f k8s/service.yaml
        }
    }
}
```

Each stage is independently logged. Failures are easy to isolate and retry.

---

## Kubernetes Concepts Covered

This project goes hands-on with the full Kubernetes workload model on Minikube:

- **Pods** — smallest deployable unit, wrapping the Docker container
- **Deployments** — declarative updates, rollback support, desired-state management
- **ReplicaSets** — maintain the specified number of pod replicas automatically
- **Services (NodePort)** — expose pods to external traffic outside the cluster
- **Minikube** — single-node cluster for zero-cost development on EC2

A companion interactive guide (`k8s/kubernetes-beginner-guide.html`) covers cluster architecture, the control plane, worker nodes, and the full request lifecycle from DNS to pod.

---

## Design Decisions

**Why Minikube instead of EKS?**
Keeps costs at zero while covering the same Kubernetes API surface. The `deployment.yaml` and `service.yaml` manifests are portable to any standard cluster — upgrading to EKS later requires no changes to the K8s config.

**Why Amazon ECR instead of Docker Hub?**
ECR integrates natively with AWS IAM. The EC2 instance role grants pull access automatically — no credentials need to be stored in Jenkins.

**Why Ansible instead of EC2 user data scripts?**
Ansible playbooks are idempotent and rerunnable. If server configuration drifts, re-running the playbook restores the desired state without re-provisioning the instance.

**Why a declarative Jenkinsfile?**
The pipeline definition lives in version control alongside the application code. Every change is tracked, reviewable, and rollback-able like any other file in the repo.

---

## Roadmap

- [ ] Migrate from Minikube to **EKS** for production-grade scaling
- [ ] Add **automated rollback** on deployment failure
- [ ] Set up **Prometheus + Grafana** monitoring dashboards
- [ ] Implement multi-environment pipelines — **dev / staging / prod**
- [ ] Move secrets to **Jenkins Credentials Manager** (no plain-text keys)
- [ ] Enable **Slack / email notifications** on pipeline success or failure

---

## Author

**Anshu Sharma** — Cloud & DevOps Engineer

Building production-grade infrastructure and automation on AWS.

---

<div align="center">⭐ Star this repo if it was useful.</div>