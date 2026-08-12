# Jenkins Zero to Hero — Lab 11

## Complete Jenkins CI/CD Pipeline

This lab combines everything we have learned so far.

```text
Lab 1  → Jenkins Installation
Lab 2  → Jenkins Jobs, Workspace and Parameters
Lab 3  → Jenkins + Git + GitHub
Lab 4  → GitHub Webhooks
Lab 5  → Jenkins Pipeline and Jenkinsfile
Lab 6  → Jenkinsfile Deep Dive
Lab 7  → Jenkins CI Pipeline
Lab 8  → Jenkins + Docker
Lab 9  → Jenkins + Docker Hub
Lab 10 → Jenkins + Kubernetes
```

In this lab, we will create a complete CI/CD pipeline:

```text
Developer
    |
    | git push
    v
GitHub
    |
    | webhook
    v
Jenkins
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
    +---- Docker Login
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
    +---- Deployment
    |
    +---- Pods
    |
    +---- Service
    |
    v
Application
```

---

# Lab Objective

By the end of this lab, you will have built a complete CI/CD pipeline that:

```text
1. Gets source code from GitHub
2. Validates the application
3. Runs tests
4. Builds a Docker image
5. Tests the Docker image
6. Logs in to Docker Hub using Jenkins Credentials
7. Pushes the Docker image to Docker Hub
8. Deploys the image to Kubernetes
9. Waits for the Kubernetes rollout
10. Verifies the deployment
11. Archives build information
```

---

# Important

We will use the existing:

```text
GitHub Repository:
jenkins-demo-app
```

Docker Hub repository:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

Kubernetes namespace:

```text
jenkins-demo
```

Jenkins job:

```text
lab-05-pipeline
```

---

# Prerequisites

Check Docker:

```bash
docker ps
```

Check Jenkins:

```bash
docker ps --filter "name=jenkins"
```

Check Docker CLI inside Jenkins:

```bash
docker exec jenkins docker --version
```

Check kubectl inside Jenkins:

```bash
docker exec jenkins kubectl version --client
```

Check Kubernetes:

```bash
kubectl get nodes
```

Check Minikube:

```bash
minikube status
```

---

# Part 1 — Check Minikube

Run:

```bash
minikube status
```

Expected:

```text
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

If Minikube is stopped:

```bash
minikube start
```

Check again:

```bash
minikube status
```

---

# Part 2 — Check Kubernetes Nodes

Run:

```bash
kubectl get nodes
```

All nodes should show:

```text
Ready
```

---

# Part 3 — Check the Kubernetes Namespace

Run:

```bash
kubectl get namespace jenkins-demo
```

Expected:

```text
jenkins-demo
```

If the namespace does not exist:

```bash
kubectl create namespace jenkins-demo
```

Check again:

```bash
kubectl get namespace jenkins-demo
```

---

# Part 4 — Check Existing Kubernetes Deployment

Run:

```bash
kubectl get deployment -n jenkins-demo
```

You should see:

```text
jenkins-demo-app
```

---

# Part 5 — Check Existing Kubernetes Pods

Run:

```bash
kubectl get pods -n jenkins-demo
```

The Pods should eventually be:

```text
Running
```

---

# Part 6 — Check Existing Kubernetes Service

Run:

```bash
kubectl get service -n jenkins-demo
```

You should see:

```text
jenkins-demo-service
```

---

# Part 7 — Go to the Application Repository

Open Git Bash.

Run:

```bash
cd /c/project/jenkins-demo-app
```

Check:

```bash
pwd
```

Check files:

```bash
ls
```

You should have:

```text
Dockerfile
Jenkinsfile
README.md
app.sh
version.txt
```

You may also have:

```text
k8s-deployment.yaml
k8s-namespace.yaml
k8s-service.yaml
```

---

# Part 8 — Check Git Status

Run:

```bash
git status
```

Expected:

```text
On branch main
nothing to commit, working tree clean
```

---

# Part 9 — Check GitHub Remote

Run:

```bash
git remote -v
```

Expected:

```text
origin  https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app.git (fetch)
origin  https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app.git (push)
```

---

# Part 10 — Create a Backup of the Existing Jenkinsfile

Run:

```bash
cp Jenkinsfile Jenkinsfile.lab10-backup
```

Check:

```bash
ls Jenkinsfile*
```

You should see:

```text
Jenkinsfile
Jenkinsfile.lab10-backup
```

---

# Part 11 — Create the Complete CI/CD Jenkinsfile

Open:

```bash
code Jenkinsfile
```

Replace the existing contents with:

```groovy
pipeline {

    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
        timeout(time: 20, unit: 'MINUTES')
    }

    parameters {

        string(
            name: 'APP_VERSION',
            defaultValue: '1.0',
            description: 'Docker image version'
        )

        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'test', 'prod'],
            description: 'Deployment environment'
        )
    }

    environment {

        DOCKERHUB_USERNAME = 'YOUR-DOCKERHUB-USERNAME'

        IMAGE_NAME = 'jenkins-demo-app'

        IMAGE = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}"

        K8S_NAMESPACE = 'jenkins-demo'

        K8S_DEPLOYMENT = 'jenkins-demo-app'

        K8S_CONTAINER = 'jenkins-demo-app'
    }

    stages {

        stage('Checkout') {

            steps {

                echo '======================================'
                echo 'STAGE 1: CHECKOUT'
                echo '======================================'

                checkout scm

                sh 'git branch --show-current'

                sh 'git log --oneline -1'
            }
        }

        stage('Validate') {

            steps {

                echo '======================================'
                echo 'STAGE 2: VALIDATE'
                echo '======================================'

                sh 'test -f Jenkinsfile'
                sh 'test -f Dockerfile'
                sh 'test -f app.sh'
                sh 'test -f version.txt'
                sh 'test -f README.md'

                echo 'Required files exist.'

                sh 'test -s Dockerfile'
                sh 'test -s app.sh'
                sh 'test -s version.txt'

                echo 'Required files are not empty.'
            }
        }

        stage('Application Test') {

            steps {

                echo '======================================'
                echo 'STAGE 3: APPLICATION TEST'
                echo '======================================'

                sh 'chmod +x app.sh'

                sh 'grep -q "JENKINS DEMO APPLICATION" app.sh'

                sh 'grep -q "Jenkins Demo App" app.sh'

                sh 'grep -q "Application is running successfully." app.sh'

                echo 'Application tests passed.'
            }
        }

        stage('Docker Build') {

            steps {

                echo '======================================'
                echo 'STAGE 4: DOCKER BUILD'
                echo '======================================'

                sh 'docker version'

                sh 'docker build -t "$IMAGE:$APP_VERSION" .'

                sh 'docker image inspect "$IMAGE:$APP_VERSION" > /dev/null'

                echo "Docker image created:"
                echo "${IMAGE}:${params.APP_VERSION}"
            }
        }

        stage('Docker Test') {

            steps {

                echo '======================================'
                echo 'STAGE 5: DOCKER TEST'
                echo '======================================'

                sh 'docker run --rm "$IMAGE:$APP_VERSION"'

                echo 'Docker container test passed.'
            }
        }

        stage('Docker Login') {

            steps {

                echo '======================================'
                echo 'STAGE 6: DOCKER LOGIN'
                echo '======================================'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_TOKEN" | docker login \
                            --username "$DOCKER_USER" \
                            --password-stdin
                    '''
                }

                echo 'Docker Hub login completed.'
            }
        }

        stage('Docker Push') {

            steps {

                echo '======================================'
                echo 'STAGE 7: DOCKER PUSH'
                echo '======================================'

                sh 'docker push "$IMAGE:$APP_VERSION"'

                echo "Docker image pushed:"
                echo "${IMAGE}:${params.APP_VERSION}"
            }
        }

        stage('Kubernetes Deploy') {

            steps {

                echo '======================================'
                echo 'STAGE 8: KUBERNETES DEPLOY'
                echo '======================================'

                sh 'kubectl version --client'

                sh 'kubectl get namespace "$K8S_NAMESPACE"'

                sh 'kubectl set image deployment/"$K8S_DEPLOYMENT" "$K8S_CONTAINER"="$IMAGE:$APP_VERSION" -n "$K8S_NAMESPACE"'

                sh 'kubectl rollout status deployment/"$K8S_DEPLOYMENT" -n "$K8S_NAMESPACE" --timeout=120s'
            }
        }

        stage('Kubernetes Verify') {

            steps {

                echo '======================================'
                echo 'STAGE 9: KUBERNETES VERIFY'
                echo '======================================'

                sh 'kubectl get deployment "$K8S_DEPLOYMENT" -n "$K8S_NAMESPACE"'

                sh 'kubectl get pods -n "$K8S_NAMESPACE" -l app="$K8S_DEPLOYMENT" -o wide'

                sh 'kubectl get service -n "$K8S_NAMESPACE"'

                sh 'kubectl get deployment "$K8S_DEPLOYMENT" -n "$K8S_NAMESPACE" -o jsonpath="{.spec.template.spec.containers[0].image}"'

                echo ''

                sh 'kubectl rollout status deployment/"$K8S_DEPLOYMENT" -n "$K8S_NAMESPACE" --timeout=120s'
            }
        }

        stage('Create Build Information') {

            steps {

                echo '======================================'
                echo 'STAGE 10: BUILD INFORMATION'
                echo '======================================'

                sh 'rm -rf build'

                sh 'mkdir -p build'

                sh 'echo "Application=$IMAGE_NAME" > build/build-info.txt'

                sh 'echo "DockerImage=$IMAGE:$APP_VERSION" >> build/build-info.txt'

                sh 'echo "Environment=$ENVIRONMENT" >> build/build-info.txt'

                sh 'echo "JenkinsBuild=$BUILD_NUMBER" >> build/build-info.txt'

                sh 'echo "GitCommit=$(git rev-parse HEAD)" >> build/build-info.txt'

                sh 'echo "GitBranch=$(git branch --show-current)" >> build/build-info.txt'

                sh 'echo "KubernetesNamespace=$K8S_NAMESPACE" >> build/build-info.txt'

                sh 'echo "KubernetesDeployment=$K8S_DEPLOYMENT" >> build/build-info.txt'

                sh 'cat build/build-info.txt'
            }
        }

        stage('Archive') {

            steps {

                echo '======================================'
                echo 'STAGE 11: ARCHIVE'
                echo '======================================'

                archiveArtifacts artifacts: 'build/**', fingerprint: true

                echo 'Build information archived.'
            }
        }
    }

    post {

        success {

            echo '======================================'
            echo 'CI/CD PIPELINE SUCCESS'
            echo '======================================'

            echo "Application: ${IMAGE_NAME}"

            echo "Docker Image: ${IMAGE}:${params.APP_VERSION}"

            echo "Environment: ${params.ENVIRONMENT}"

            echo "Kubernetes Namespace: ${K8S_NAMESPACE}"

            echo "Kubernetes Deployment: ${K8S_DEPLOYMENT}"
        }

        failure {

            echo '======================================'
            echo 'CI/CD PIPELINE FAILED'
            echo '======================================'

            echo 'Check the failed stage and Console Output.'
        }

        always {

            echo '======================================'
            echo 'PIPELINE FINISHED'
            echo '======================================'
        }
    }
}
```

Replace:

```text
YOUR-DOCKERHUB-USERNAME
```

with your actual Docker Hub username.

For example:

```groovy
DOCKERHUB_USERNAME = 'mydockeruser'
```

Do not put your Docker Hub password or token into the Jenkinsfile.

---

# Part 12 — Check the Jenkinsfile

Run:

```bash
cat Jenkinsfile
```

You should see these stages:

```text
Checkout
Validate
Application Test
Docker Build
Docker Test
Docker Login
Docker Push
Kubernetes Deploy
Kubernetes Verify
Create Build Information
Archive
```

---

# Part 13 — Validate the Application Files

Run:

```bash
test -f Dockerfile && echo "Dockerfile exists"
```

Expected:

```text
Dockerfile exists
```

Run:

```bash
test -f Jenkinsfile && echo "Jenkinsfile exists"
```

Expected:

```text
Jenkinsfile exists
```

Run:

```bash
test -f app.sh && echo "app.sh exists"
```

Expected:

```text
app.sh exists
```

Run:

```bash
test -f version.txt && echo "version.txt exists"
```

Expected:

```text
version.txt exists
```

Run:

```bash
test -f README.md && echo "README.md exists"
```

Expected:

```text
README.md exists
```

---

# Part 14 — Test the Application Locally

Make the script executable:

```bash
chmod +x app.sh
```

Run:

```bash
./app.sh
```

Expected output:

```text
======================================
JENKINS DEMO APPLICATION
======================================

Application: Jenkins Demo App
Environment: Jenkins CI
Version: <your-version>

Application is running successfully.

======================================
```

---

# Part 15 — Test the Docker Build Locally

Build a test image:

```bash
docker build -t jenkins-demo-app:cicd-test .
```

Check:

```bash
docker images | grep jenkins-demo-app
```

Expected:

```text
jenkins-demo-app   cicd-test
```

Run:

```bash
docker run --rm jenkins-demo-app:cicd-test
```

The application should run successfully.

---

# Part 16 — Remove the Local Test Image

Run:

```bash
docker rmi jenkins-demo-app:cicd-test
```

Check:

```bash
docker images | grep jenkins-demo-app
```

---

# Part 17 — Check Jenkins Credentials

Before running the full pipeline, verify that this Jenkins credential exists:

```text
dockerhub-credentials
```

Open:

```text
Jenkins
→ Manage Jenkins
→ Credentials
→ Global
```

You should see:

```text
dockerhub-credentials
```

---

# Part 18 — Check Jenkins Kubernetes Access

From Git Bash:

```bash
docker exec jenkins kubectl get nodes
```

You should see your Minikube node or nodes.

Check namespace:

```bash
docker exec jenkins kubectl get namespace jenkins-demo
```

Check Deployment:

```bash
docker exec jenkins kubectl get deployment -n jenkins-demo
```

---

# Part 19 — Check Jenkins Docker Access

Run:

```bash
docker exec jenkins docker ps
```

Jenkins should be able to communicate with Docker.

---

# Part 20 — Check Docker Hub Access Manually

Do not put the token into the command.

The actual Docker Hub authentication will be handled by Jenkins Credentials during the pipeline.

Make sure your Docker Hub repository exists:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

---

# Part 21 — Check Git Status

Run:

```bash
git status
```

Expected:

```text
modified: Jenkinsfile
```

---

# Part 22 — Commit the Complete CI/CD Pipeline

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Build complete Jenkins CI/CD pipeline"
```

---

# Part 23 — Push the Pipeline

Run:

```bash
git push origin main
```

---

# Part 24 — Open Jenkins

Open:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

Make sure:

```text
Definition:
Pipeline script from SCM
```

SCM:

```text
Git
```

Repository:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app.git
```

Branch:

```text
*/main
```

Script Path:

```text
Jenkinsfile
```

---

# Part 25 — Run the Complete Pipeline

Click:

```text
Build with Parameters
```

Enter:

```text
APP_VERSION:
20.0
```

Choose:

```text
ENVIRONMENT:
dev
```

Click:

```text
Build
```

---

# Part 26 — Watch the Pipeline

The pipeline should execute:

```text
Checkout
    |
    v
Validate
    |
    v
Application Test
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
Kubernetes Deploy
    |
    v
Kubernetes Verify
    |
    v
Build Information
    |
    v
Archive
```

---

# Part 27 — Check Console Output

Open the latest build.

Click:

```text
Console Output
```

Look for:

```text
STAGE 1: CHECKOUT
```

Then:

```text
STAGE 2: VALIDATE
```

Then:

```text
STAGE 3: APPLICATION TEST
```

Then:

```text
STAGE 4: DOCKER BUILD
```

Then:

```text
STAGE 5: DOCKER TEST
```

Then:

```text
STAGE 6: DOCKER LOGIN
```

Then:

```text
STAGE 7: DOCKER PUSH
```

Then:

```text
STAGE 8: KUBERNETES DEPLOY
```

Then:

```text
STAGE 9: KUBERNETES VERIFY
```

Then:

```text
STAGE 10: BUILD INFORMATION
```

Then:

```text
STAGE 11: ARCHIVE
```

Finally:

```text
CI/CD PIPELINE SUCCESS
```

---

# Part 28 — Verify Docker Hub Image

Open Docker Hub.

Go to:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

You should now see:

```text
20.0
```

---

# Part 29 — Verify Docker Image Locally

Run:

```bash
docker images | grep jenkins-demo-app
```

You should see:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app   20.0
```

---

# Part 30 — Run the Docker Image

Run:

```bash
docker run --rm YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:20.0
```

The application should run successfully.

---

# Part 31 — Verify Kubernetes Deployment

Run:

```bash
kubectl get deployment -n jenkins-demo
```

Expected:

```text
jenkins-demo-app
```

---

# Part 32 — Verify Kubernetes Pods

Run:

```bash
kubectl get pods -n jenkins-demo
```

The Pods should be:

```text
Running
```

---

# Part 33 — Check Pods in Detail

Run:

```bash
kubectl get pods -n jenkins-demo -o wide
```

You should see:

```text
NAME
READY
STATUS
RESTARTS
AGE
IP
NODE
```

---

# Part 34 — Verify the Kubernetes Image

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:20.0
```

---

# Part 35 — Verify Deployment Rollout

Run:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

Expected:

```text
deployment "jenkins-demo-app" successfully rolled out
```

---

# Part 36 — Verify Kubernetes Service

Run:

```bash
kubectl get service -n jenkins-demo
```

Expected:

```text
jenkins-demo-service
```

---

# Part 37 — Check Kubernetes Resources

Run:

```bash
kubectl get all -n jenkins-demo
```

You should see:

```text
deployment
replicaset
pods
service
```

---

# Part 38 — Check Build Information

Open the Jenkins build.

Find:

```text
Build Artifacts
```

Open:

```text
build/
```

Then:

```text
build-info.txt
```

Expected format:

```text
Application=jenkins-demo-app
DockerImage=YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:20.0
Environment=dev
JenkinsBuild=<build-number>
GitCommit=<commit-hash>
GitBranch=main
KubernetesNamespace=jenkins-demo
KubernetesDeployment=jenkins-demo-app
```

---

# Part 39 — Test the Complete Pipeline Again

Now use:

```text
APP_VERSION:
21.0
```

Environment:

```text
test
```

Run:

```text
Build with Parameters
```

Click:

```text
Build
```

Wait for the pipeline to complete.

---

# Part 40 — Verify Version 21.0 in Docker Hub

Open Docker Hub.

You should see:

```text
20.0
21.0
```

---

# Part 41 — Verify Version 21.0 in Kubernetes

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:21.0
```

---

# Part 42 — Verify Pods After Deployment

Run:

```bash
kubectl get pods -n jenkins-demo
```

The Pods should be:

```text
Running
```

Check rollout:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

---

# Part 43 — Test Production Version

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
22.0
```

Choose:

```text
ENVIRONMENT:
prod
```

Click:

```text
Build
```

---

# Part 44 — Verify Production Deployment

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:22.0
```

---

# Part 45 — Understand the Complete CI/CD Pipeline

The complete process is now:

```text
                Developer
                    |
                    | git push
                    v
                 GitHub
                    |
                    | webhook
                    v
                 Jenkins
                    |
              +-----+------+
              |            |
              v            v
           Checkout      Jenkinsfile
              |
              v
           Validate
              |
              v
        Application Test
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
      Kubernetes Deploy
              |
              v
      Kubernetes Verify
              |
              v
           Pods Running
```

---

# Part 46 — Understand CI vs CD

Continuous Integration:

```text
GitHub
   |
   v
Jenkins
   |
   +---- Build
   +---- Test
   +---- Package
```

Continuous Delivery / Deployment:

```text
Docker Image
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

We have now combined both.

---

# Part 47 — Test Automatic Trigger

If the GitHub webhook from Lab 4 is active, make a small Git change.

Open:

```bash
code README.md
```

Add:

```text
Complete Jenkins CI/CD pipeline tested.
```

Save.

---

# Part 48 — Commit the Change

Run:

```bash
git add README.md
```

Commit:

```bash
git commit -m "Test complete CI/CD pipeline trigger"
```

---

# Part 49 — Push the Change

Run:

```bash
git push origin main
```

Expected flow:

```text
Git Push
    |
    v
GitHub
    |
    v
Webhook
    |
    v
Jenkins
    |
    v
Complete CI/CD Pipeline
```

---

# Part 50 — Check Jenkins Build History

Open:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

Check:

```text
Build History
```

A new build should appear if your webhook and Cloudflare tunnel are running.

If the webhook is not active:

```text
Build with Parameters
```

can still be used to run the pipeline manually.

---

# Part 51 — Check Latest Git Commit

In Jenkins console output, look for:

```text
git log --oneline -1
```

You should see:

```text
Test complete CI/CD pipeline trigger
```

---

# Part 52 — Verify Kubernetes After Git Push

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo
```

Then:

```bash
kubectl get pods -n jenkins-demo
```

Then:

```bash
kubectl get service -n jenkins-demo
```

---

# Part 53 — Test a Pipeline Failure

We will intentionally break the pipeline.

Open:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

Inside:

```text
Validate
```

add:

```groovy
sh 'test -f this-file-does-not-exist.txt'
```

Save.

---

# Part 54 — Commit the Broken Pipeline

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Test complete CI/CD pipeline failure"
```

Push:

```bash
git push origin main
```

---

# Part 55 — Run the Failed Pipeline

Open Jenkins.

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
23.0
```

Environment:

```text
dev
```

Click:

```text
Build
```

The pipeline should fail at:

```text
Validate
```

---

# Part 56 — Check the Failure

Open:

```text
Console Output
```

You should see an error related to:

```text
this-file-does-not-exist.txt
```

The pipeline should not continue to:

```text
Docker Build
Docker Push
Kubernetes Deploy
```

This is important.

A broken application should not be deployed.

---

# Part 57 — Fix the Pipeline

Open:

```bash
code Jenkinsfile
```

Remove:

```groovy
sh 'test -f this-file-does-not-exist.txt'
```

Save.

---

# Part 58 — Commit the Fix

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Fix complete CI/CD validation"
```

Push:

```bash
git push origin main
```

---

# Part 59 — Run the Fixed Pipeline

Open Jenkins.

Click:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
24.0
```

Environment:

```text
dev
```

Click:

```text
Build
```

The pipeline should now succeed.

---

# Part 60 — Verify Version 24.0

Docker Hub:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:24.0
```

Kubernetes:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:24.0
```

---

# Part 61 — Check Kubernetes Rollout

Run:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

Expected:

```text
deployment "jenkins-demo-app" successfully rolled out
```

---

# Part 62 — Check Kubernetes Pods

Run:

```bash
kubectl get pods -n jenkins-demo -o wide
```

Expected:

```text
Running
```

---

# Part 63 — Check Deployment History

Run:

```bash
kubectl rollout history deployment/jenkins-demo-app -n jenkins-demo
```

You should see multiple revisions.

---

# Part 64 — Test Kubernetes Rollback

Check current image:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Then:

```bash
kubectl rollout undo deployment/jenkins-demo-app -n jenkins-demo
```

Check:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

Check the image:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

The Deployment should have returned to its previous revision.

---

# Part 65 — Restore the Desired Version Through Jenkins

Jenkins should remain the source of the deployment process.

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
25.0
```

Choose:

```text
ENVIRONMENT:
dev
```

Click:

```text
Build
```

After success:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:25.0
```

---

# Part 66 — Check Everything

Docker:

```bash
docker ps
```

Jenkins:

```bash
docker ps --filter "name=jenkins"
```

Docker image:

```bash
docker images | grep jenkins-demo-app
```

Kubernetes nodes:

```bash
kubectl get nodes
```

Kubernetes deployment:

```bash
kubectl get deployment -n jenkins-demo
```

Kubernetes Pods:

```bash
kubectl get pods -n jenkins-demo
```

Kubernetes service:

```bash
kubectl get service -n jenkins-demo
```

---

# Part 67 — Complete Architecture

Your complete project now looks like:

```text
                         GitHub
                           |
                           | git push
                           v
                    GitHub Webhook
                           |
                           v
                        Jenkins
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

# Part 68 — Why This Is a Real DevOps Project

You have connected:

```text
Git
GitHub
Jenkins
Docker
Docker Hub
Kubernetes
Minikube
kubectl
CI
CD
Webhooks
Credentials
Artifacts
Rolling Deployments
Rollback
```

The entire path is:

```text
Source Code
    |
    v
GitHub
    |
    v
Jenkins
    |
    v
Docker Image
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

# Part 69 — Check Git Status

Go to:

```bash
cd /c/project/jenkins-demo-app
```

Run:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# Part 70 — Check Git History

Run:

```bash
git log --oneline --max-count=10
```

You should see commits related to:

```text
Build complete Jenkins CI/CD pipeline
Test complete CI/CD pipeline trigger
Test complete CI/CD pipeline failure
Fix complete CI/CD validation
```

Your exact history will be different.

---

# Part 71 — Save Lab 11 Documentation

Go to the Jenkins documentation repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go to labs:

```bash
cd labs
```

Create:

```bash
touch lab-11-complete-cicd-pipeline.md
```

Open:

```bash
code lab-11-complete-cicd-pipeline.md
```

Paste this entire lab into the file.

Save.

---

# Part 72 — Check Git Status

Return to the repository root:

```bash
cd /c/project/jenkins-zero-to-hero
```

Run:

```bash
git status
```

You should see:

```text
new file: labs/lab-11-complete-cicd-pipeline.md
```

---

# Part 73 — Add Lab 11

```bash
git add labs/lab-11-complete-cicd-pipeline.md
```

Check:

```bash
git status
```

---

# Part 74 — Commit Lab 11

```bash
git commit -m "Add Jenkins Lab 11 complete CI/CD pipeline"
```

---

# Part 75 — Push Lab 11

```bash
git push origin main
```

Check:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# Part 76 — Verify Repository Structure

Your Jenkins documentation repository should now contain:

```text
jenkins-zero-to-hero/
│
├── README.md
│
└── labs/
    │
    ├── lab-01-install-jenkins.md
    │
    ├── lab-02-jenkins-jobs-workspace.md
    │
    ├── lab-03-jenkins-git-github.md
    │
    ├── lab-04-jenkins-webhooks.md
    │
    ├── lab-05-jenkins-pipeline.md
    │
    ├── lab-06-jenkinsfile-deep-dive.md
    │
    ├── lab-07-jenkins-ci-pipeline.md
    │
    ├── lab-08-jenkins-docker.md
    │
    ├── lab-09-jenkins-dockerhub.md
    │
    ├── lab-10-jenkins-kubernetes.md
    │
    └── lab-11-complete-cicd-pipeline.md
```

---

# Lab 11 Completion Checklist

```text
[ ] Jenkins is running
[ ] Docker is accessible from Jenkins
[ ] kubectl is accessible from Jenkins
[ ] Minikube is running
[ ] Kubernetes namespace verified
[ ] Kubernetes Deployment verified
[ ] Kubernetes Service verified
[ ] GitHub repository verified
[ ] Docker Hub repository verified
[ ] Jenkins Docker Hub credential verified
[ ] Complete Jenkinsfile created
[ ] Checkout stage completed
[ ] Validate stage completed
[ ] Application Test completed
[ ] Docker Build completed
[ ] Docker Test completed
[ ] Docker Login completed
[ ] Docker Push completed
[ ] Docker Hub image verified
[ ] Kubernetes Deploy completed
[ ] Kubernetes Verify completed
[ ] Build information created
[ ] Build artifact archived
[ ] Kubernetes image verified
[ ] Multiple application versions deployed
[ ] GitHub webhook tested
[ ] Automatic pipeline tested
[ ] Pipeline failure tested
[ ] Pipeline failure fixed
[ ] Kubernetes rollback tested
[ ] Jenkins redeployed the application
[ ] Final Kubernetes Pods verified
[ ] Git status clean
[ ] Lab 11 documentation saved
[ ] Lab 11 committed
[ ] Lab 11 pushed to GitHub
```

---

# Lab 11 Cleanup

## Recommended

If you are continuing to Lab 12, keep:

```text
jenkins
jenkins_home
jenkins network
jenkins-demo-app
Docker Hub repository
Minikube
Kubernetes namespace
```

We will reuse them.

---

# Delete Kubernetes Resources

If you want to clean only the application:

```bash
kubectl delete deployment jenkins-demo-app -n jenkins-demo
```

Delete the Service:

```bash
kubectl delete service jenkins-demo-service -n jenkins-demo
```

Check:

```bash
kubectl get all -n jenkins-demo
```

---

# Delete the Namespace

To remove everything in the namespace:

```bash
kubectl delete namespace jenkins-demo
```

Verify:

```bash
kubectl get namespaces
```

---

# Recreate the Namespace

If you removed it and need it again:

```bash
kubectl create namespace jenkins-demo
```

---

# Remove Docker Images

Check:

```bash
docker images | grep jenkins-demo-app
```

Remove a specific image:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:20.0
```

Remove another:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:21.0
```

Remove another:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:22.0
```

Only remove versions you no longer need.

---

# Stop Minikube

Only do this if you are finished using Kubernetes for now:

```bash
minikube stop
```

Check:

```bash
minikube status
```

---

# Start Minikube Again

When continuing:

```bash
minikube start
```

Check:

```bash
minikube status
```

---

# Complete Jenkins Cleanup

Only use these commands if you intentionally want to delete Jenkins completely.

Stop:

```bash
docker stop jenkins
```

Remove:

```bash
docker rm jenkins
```

Remove Jenkins data:

```bash
docker volume rm jenkins_home
```

Remove network:

```bash
docker network rm jenkins
```

Verify:

```bash
docker ps -a
```

Check volumes:

```bash
docker volume ls
```

Check networks:

```bash
docker network ls
```

---

# What You Learned

Lab 1:

```text
Jenkins Installation
Docker
Jenkins Dashboard
First Job
First Build
```

Lab 2:

```text
Jobs
Build History
Workspace
Environment Variables
Parameters
Artifacts
```

Lab 3:

```text
Git
GitHub
Git Checkout
Jenkins Git Integration
```

Lab 4:

```text
GitHub Webhooks
Automatic Jenkins Trigger
Git Push → Jenkins
```

Lab 5:

```text
Jenkins Pipeline
Jenkinsfile
Declarative Pipeline
Stages
Steps
Parameters
Artifacts
```

Lab 6:

```text
Advanced Jenkinsfile
Agent
Environment
Options
Script
When
Post
Conditional Stages
Failure Handling
```

Lab 7:

```text
Continuous Integration
Checkout
Validation
Build
Testing
Packaging
Artifact Creation
Build Traceability
```

Lab 8:

```text
Jenkins + Docker
Custom Jenkins Image
Docker CLI
Docker Socket
Dockerfile
Docker Build
Docker Run
Docker Test
Docker Image Tags
```

Lab 9:

```text
Docker Hub
Docker Registry
Docker Repository
Docker Hub PAT
Jenkins Credentials
Docker Login
Docker Push
Docker Pull
```

Lab 10:

```text
Jenkins + Kubernetes
kubectl
Minikube
Namespace
Deployment
ReplicaSet
Pods
Service
Rolling Update
Rollout
Rollback
```

Lab 11:

```text
Complete CI/CD
GitHub
Webhook
Jenkins
Jenkinsfile
Docker
Docker Hub
Kubernetes
Automated Deployment
Deployment Verification
Rollback
CI Failure
CI Recovery
```

---

# Next Lab

## Lab 12 — Jenkins Credentials

In the next lab, we will focus specifically on Jenkins credentials and secure secret handling.

We will learn:

```text
Jenkins Credentials
Username + Password
Secret Text
SSH Keys
GitHub Token
Docker Hub Token
Credential IDs
withCredentials
Credentials Binding
Environment Variables
Secret Masking
```

The goal is to understand how real Jenkins pipelines handle passwords, tokens and other secrets without putting them inside GitHub.

# End of Lab 11
