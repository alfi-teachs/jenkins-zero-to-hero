# Jenkins Zero to Hero

A hands-on Jenkins learning project that takes you from running your first Jenkins container to building a complete CI/CD pipeline with GitHub, Docker, Docker Hub, Kubernetes, Jenkins Agents, Shared Libraries, Credentials, and Security.

---

## 🚀 What You Will Build

By the end of this project, the complete workflow will look like:

```text
Developer
    |
    | git push
    v
GitHub
    |
    | Webhook
    v
Jenkins
    |
    v
Jenkins Agent
    |
    +---- Checkout
    |
    +---- Validate
    |
    +---- Test
    |
    +---- Docker Build
    |
    +---- Docker Test
    |
    +---- Docker Push
    |
    +---- Kubernetes Deploy
    |
    +---- Kubernetes Verify
    |
    v
Docker Hub
    |
    v
Kubernetes
    |
    v
Application
```

---

# 📚 Labs

Click any lab below to open the complete hands-on guide.

| # | Lab | Topics |
|---|---|---|
| 01 | [Install Jenkins](labs/lab-01-install-jenkins.md) | Jenkins, Docker, Jenkins Dashboard, First Job |
| 02 | [Jenkins Jobs, Workspace & Parameters](labs/lab-02-jenkins-jobs-workspace.md) | Jobs, Builds, Workspace, Environment Variables, Parameters, Artifacts |
| 03 | [Jenkins + Git + GitHub](labs/lab-03-jenkins-git-github.md) | Git, GitHub, Git Checkout, Jenkins Git Integration |
| 04 | [Jenkins + GitHub Webhooks](labs/lab-04-jenkins-webhooks.md) | Webhooks, Automatic Builds, Cloudflare Tunnel |
| 05 | [Jenkins Pipeline](labs/lab-05-jenkins-pipeline.md) | Pipeline, Jenkinsfile, Stages, Steps, Parameters |
| 06 | [Jenkinsfile Deep Dive](labs/lab-06-jenkinsfile-deep-dive.md) | Agent, Environment, Options, Script, When, Post |
| 07 | [Jenkins CI Pipeline](labs/lab-07-jenkins-ci-pipeline.md) | Continuous Integration, Build, Test, Package, Archive |
| 08 | [Jenkins + Docker](labs/lab-08-jenkins-docker.md) | Docker CLI, Dockerfile, Docker Build, Docker Test |
| 09 | [Jenkins + Docker Hub](labs/lab-09-jenkins-dockerhub.md) | Docker Hub, Registry, PAT, Credentials, Docker Push |
| 10 | [Jenkins + Kubernetes](labs/lab-10-jenkins-kubernetes.md) | Minikube, kubectl, Deployment, Pods, Service, Rollout |
| 11 | [Complete Jenkins CI/CD Pipeline](labs/lab-11-complete-cicd-pipeline.md) | GitHub, Jenkins, Docker, Docker Hub, Kubernetes |
| 12 | [Jenkins Credentials](labs/lab-12-jenkins-credentials.md) | Secrets, Credentials, Tokens, SSH Keys, withCredentials |
| 13 | [Jenkins Agents](labs/lab-13-jenkins-agents.md) | Controller, Agent, Node, Executor, Labels, Distributed Builds |
| 14 | [Jenkins Shared Libraries](labs/lab-14-jenkins-shared-libraries.md) | Shared Libraries, vars/, @Library, Reusable Functions |
| 15 | [Jenkins Security](labs/lab-15-jenkins-security.md) | Authentication, Authorization, Permissions, CSRF, Least Privilege |
| 16 | [Final CI/CD Project](labs/lab-16-final-cicd-project.md) | Complete Production-Style Jenkins CI/CD |

---

# 🛠️ Technologies Used

## Jenkins

```text
Jenkins
Jenkinsfile
Declarative Pipeline
Jenkins Agents
Shared Libraries
Credentials
Security
```

## Source Control

```text
Git
GitHub
GitHub Webhooks
```

## Containers

```text
Docker
Dockerfile
Docker Hub
Docker Registry
Docker Images
Docker Containers
```

## Kubernetes

```text
Kubernetes
Minikube
kubectl
Deployments
ReplicaSets
Pods
Services
Rolling Updates
Rollback
```

---

# 📁 Repository Structure

```text
jenkins-zero-to-hero/
│
├── README.md
│
└── labs/
    │
    ├── lab-01-install-jenkins.md
    ├── lab-02-jenkins-jobs-workspace.md
    ├── lab-03-jenkins-git-github.md
    ├── lab-04-jenkins-webhooks.md
    ├── lab-05-jenkins-pipeline.md
    ├── lab-06-jenkinsfile-deep-dive.md
    ├── lab-07-jenkins-ci-pipeline.md
    ├── lab-08-jenkins-docker.md
    ├── lab-09-jenkins-dockerhub.md
    ├── lab-10-jenkins-kubernetes.md
    ├── lab-11-complete-cicd-pipeline.md
    ├── lab-12-jenkins-credentials.md
    ├── lab-13-jenkins-agents.md
    ├── lab-14-jenkins-shared-libraries.md
    ├── lab-15-jenkins-security.md
    └── lab-16-final-cicd-project.md
```

---

# 🧪 Learning Path

The labs are designed to be completed in order:

```text
Jenkins
   ↓
Jobs
   ↓
Git + GitHub
   ↓
Webhooks
   ↓
Pipeline
   ↓
CI
   ↓
Docker
   ↓
Docker Hub
   ↓
Kubernetes
   ↓
Credentials
   ↓
Agents
   ↓
Shared Libraries
   ↓
Security
   ↓
Final CI/CD Project
```

---

# 🎯 Final Project Architecture

The final project combines everything:

```text
                         GitHub
                            |
                            | git push
                            v
                     GitHub Webhook
                            |
                            v
                    Jenkins Controller
                            |
                            v
                       Jenkins Agent
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
          Checkout       Validate       Test
                            |
                            v
                       Docker Build
                            |
                            v
                        Docker Test
                            |
                            v
                       Docker Login
                            |
                            v
                       Docker Push
                            |
                            v
                        Docker Hub
                            |
                            v
                  Kubernetes Deployment
                            |
                            v
                         Pods
                            |
                            v
                       Application
```

---

# 🔐 Security Practices Covered

This project also covers Jenkins security fundamentals:

```text
Authentication
Authorization
Users
Permissions
Credentials
Secret Text
Username + Password
SSH Keys
CSRF Protection
Least Privilege
Docker Socket Security
Kubernetes Access
```

Never commit:

```text
Passwords
API Keys
Docker Hub Tokens
GitHub Tokens
Private SSH Keys
Kubernetes Secrets
```

to GitHub.

---

# ✅ Skills Covered

After completing all labs, you will have hands-on practice with:

```text
Jenkins Installation
Jenkins Jobs
Jenkins Workspace
Environment Variables
Build Parameters
Freestyle Projects
Jenkins Pipelines
Jenkinsfile
Declarative Pipeline
Git Integration
GitHub Integration
GitHub Webhooks
Continuous Integration
Docker Builds
Docker Testing
Docker Hub
Docker Registry
Jenkins Credentials
Kubernetes Deployments
Kubernetes Services
Rolling Updates
Rollbacks
Jenkins Agents
Distributed Builds
Jenkins Shared Libraries
Jenkins Security
```

---

# 📌 Related Repositories

This Jenkins project uses the following repositories during the labs:

### Application Repository

```text
jenkins-demo-app
```

Contains:

```text
Application code
Dockerfile
Jenkinsfile
Kubernetes configuration
```

### Shared Library Repository

```text
jenkins-shared-library
```

Contains reusable Jenkins Pipeline functions:

```text
buildApp()
testApp()
dockerBuild()
dockerTest()
dockerPush()
k8sDeploy()
k8sVerify()
```

### Jenkins Training Repository

```text
jenkins-zero-to-hero
```

Contains all lab documentation.

---

# 🧹 Cleanup

Each lab contains its own cleanup section.

The general cleanup commands are:

```bash
docker stop jenkins
```

```bash
docker rm jenkins
```

```bash
docker volume rm jenkins_home
```

```bash
docker network rm jenkins
```

For Kubernetes:

```bash
kubectl delete namespace jenkins-demo
```

> **Warning:** Removing `jenkins_home` deletes Jenkins jobs, plugins, configuration, credentials, and build history stored in that volume.

---

# 📖 Recommended Order

Start here:

### [Lab 1 — Install Jenkins](labs/lab-01-install-jenkins.md)

Then work through the labs sequentially:

```text
01 → 02 → 03 → 04 → 05 → 06 → 07 → 08
→ 09 → 10 → 11 → 12 → 13 → 14 → 15 → 16
```

---

# 🏁 Completion

When all 16 labs are complete, you will have built:

```text
GitHub
   ↓
Webhook
   ↓
Jenkins
   ↓
Jenkins Agent
   ↓
Shared Library
   ↓
CI Pipeline
   ↓
Docker
   ↓
Docker Hub
   ↓
Kubernetes
   ↓
Application
```

---

# 👩‍💻 Project Status

```text
Labs Completed: 16 / 16
Status: Jenkins Zero to Hero Complete
```

---

# ⭐ Final Project

## GitHub → Jenkins → Docker → Docker Hub → Kubernetes

This repository demonstrates hands-on experience with:

```text
CI/CD
Jenkins
Git
GitHub
Docker
Docker Hub
Kubernetes
Minikube
Jenkins Agents
Shared Libraries
Credentials
Security
```

---

## Jenkins Zero to Hero

**From your first Jenkins job to a complete CI/CD pipeline.**