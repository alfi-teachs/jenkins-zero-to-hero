# Jenkins Zero to Hero — Lab 16

## Final Production-Style Jenkins CI/CD Project

This is the final major lab in the Jenkins Zero to Hero series.

You have completed:

```text
Lab 1  → Jenkins Installation
Lab 2  → Jobs, Workspace and Parameters
Lab 3  → Jenkins + Git + GitHub
Lab 4  → GitHub Webhooks
Lab 5  → Jenkins Pipeline and Jenkinsfile
Lab 6  → Jenkinsfile Deep Dive
Lab 7  → Jenkins CI Pipeline
Lab 8  → Jenkins + Docker
Lab 9  → Jenkins + Docker Hub
Lab 10 → Jenkins + Kubernetes
Lab 11 → Complete CI/CD Pipeline
Lab 12 → Jenkins Credentials
Lab 13 → Jenkins Agents
Lab 14 → Jenkins Shared Libraries
Lab 15 → Jenkins Security
```

Now we will combine the major concepts into one production-style project.

The final architecture is:

```text
Developer
    |
    | git push
    v
GitHub
    |
    | Webhook
    v
Jenkins Controller
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
Kubernetes / Minikube
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

# Final Project Objective

By the end of this lab, you will have a complete Jenkins CI/CD project using:

```text
Git
GitHub
GitHub Webhook
Jenkins Controller
Jenkins Agent
Jenkinsfile
Jenkins Shared Library
Jenkins Credentials
Docker
Docker Hub
Kubernetes
Minikube
kubectl
CI/CD
Deployment
Rolling Update
Rollback
```

The final pipeline will automatically:

```text
1. Receive a GitHub push
2. Start Jenkins
3. Run on a Jenkins Docker agent
4. Checkout the code
5. Validate the project
6. Run application tests
7. Build a Docker image
8. Test the Docker image
9. Authenticate to Docker Hub securely
10. Push the Docker image
11. Deploy the image to Kubernetes
12. Wait for rollout
13. Verify Pods
14. Verify Deployment
15. Create build information
16. Archive the build information
```

---

# Final Project Repositories

You now have three GitHub repositories:

```text
jenkins-zero-to-hero
```

Contains the lab documentation.

```text
jenkins-demo-app
```

Contains the application and Jenkinsfile.

```text
jenkins-shared-library
```

Contains reusable Jenkins Pipeline functions.

---

# Part 1 — Verify Docker

Open Git Bash.

Run:

```bash
docker version
```

Check containers:

```bash
docker ps
```

You should see:

```text
jenkins
```

---

# Part 2 — Verify Jenkins

Open:

```text
http://localhost:8081
```

Log in as your administrator.

---

# Part 3 — Verify Jenkins Docker Access

Run:

```bash
docker exec jenkins docker --version
```

Expected:

```text
Docker version ...
```

Check Docker engine:

```bash
docker exec jenkins docker ps
```

---

# Part 4 — Verify Jenkins kubectl Access

Run:

```bash
docker exec jenkins kubectl version --client
```

Check Kubernetes:

```bash
docker exec jenkins kubectl get nodes
```

Expected:

```text
minikube   Ready
```

or your current Minikube node configuration.

---

# Part 5 — Verify Minikube

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

# Part 6 — Verify Kubernetes Namespace

Run:

```bash
kubectl get namespace jenkins-demo
```

If it does not exist:

```bash
kubectl create namespace jenkins-demo
```

Check:

```bash
kubectl get namespace jenkins-demo
```

---

# Part 7 — Verify Docker Hub Credential

Open:

```text
Jenkins
→ Manage Jenkins
→ Credentials
→ Global
```

Verify:

```text
dockerhub-credentials
```

exists.

Do not expose the token.

---

# Part 8 — Verify Shared Library

Open:

```text
Jenkins
→ Manage Jenkins
→ System
```

Find:

```text
Global Pipeline Libraries
```

Verify:

```text
Name:
jenkins-shared-library
```

Repository:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-shared-library.git
```

Default version:

```text
main
```

---

# Part 9 — Verify Docker Agent

Open:

```text
Jenkins
→ Manage Jenkins
→ Nodes
```

Verify your Docker agent configuration.

The label should be:

```text
docker-agent
```

The agent image should be similar to:

```text
jenkins-agent:docker-1.0
```

---

# Part 10 — Verify the Shared Library Repository

Run:

```bash
cd /c/project/jenkins-shared-library
```

Check:

```bash
ls
```

You should see:

```text
README.md
resources
src
vars
```

Check functions:

```bash
find vars -maxdepth 1 -type f -print
```

Expected:

```text
vars/buildApp.groovy
vars/testApp.groovy
vars/dockerBuild.groovy
vars/dockerTest.groovy
vars/dockerPush.groovy
vars/k8sDeploy.groovy
vars/k8sVerify.groovy
```

---

# Part 11 — Verify the Application Repository

Run:

```bash
cd /c/project/jenkins-demo-app
```

Check:

```bash
pwd
```

Check:

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

# Part 12 — Check Git Status

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

# Part 13 — Check GitHub Remote

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

# Part 14 — Check the Application

Run:

```bash
cat version.txt
```

Run:

```bash
chmod +x app.sh
```

Run:

```bash
./app.sh
```

Expected:

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

# Part 15 — Check the Dockerfile

Run:

```bash
cat Dockerfile
```

Expected:

```dockerfile
FROM alpine:3.20

WORKDIR /app

COPY app.sh .
COPY version.txt .

RUN chmod +x app.sh

CMD ["./app.sh"]
```

---

# Part 16 — Check the Jenkinsfile

Run:

```bash
cat Jenkinsfile
```

The Jenkinsfile should import:

```groovy
@Library('jenkins-shared-library') _
```

and use:

```groovy
agent {
    label 'docker-agent'
}
```

It should also use functions similar to:

```text
buildApp()
testApp()
dockerBuild()
dockerTest()
dockerPush()
k8sDeploy()
k8sVerify()
```

---

# Part 17 — Create the Final Jenkinsfile

Open:

```bash
code Jenkinsfile
```

Replace the contents with:

```groovy
@Library('jenkins-shared-library') _

pipeline {

    agent {
        label 'docker-agent'
    }

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
            }
        }

        stage('Build') {

            steps {

                echo '======================================'
                echo 'STAGE 3: BUILD'
                echo '======================================'

                buildApp()
            }
        }

        stage('Test') {

            steps {

                echo '======================================'
                echo 'STAGE 4: TEST'
                echo '======================================'

                testApp()
            }
        }

        stage('Docker Build') {

            steps {

                echo '======================================'
                echo 'STAGE 5: DOCKER BUILD'
                echo '======================================'

                dockerBuild(
                    "${IMAGE}",
                    "${APP_VERSION}"
                )
            }
        }

        stage('Docker Test') {

            steps {

                echo '======================================'
                echo 'STAGE 6: DOCKER TEST'
                echo '======================================'

                dockerTest(
                    "${IMAGE}",
                    "${APP_VERSION}"
                )
            }
        }

        stage('Docker Push') {

            steps {

                echo '======================================'
                echo 'STAGE 7: DOCKER PUSH'
                echo '======================================'

                dockerPush(
                    "${IMAGE}",
                    "${APP_VERSION}",
                    'dockerhub-credentials'
                )
            }
        }

        stage('Kubernetes Deploy') {

            steps {

                echo '======================================'
                echo 'STAGE 8: KUBERNETES DEPLOY'
                echo '======================================'

                k8sDeploy(
                    "${K8S_NAMESPACE}",
                    "${K8S_DEPLOYMENT}",
                    "${K8S_CONTAINER}",
                    "${IMAGE}",
                    "${APP_VERSION}"
                )
            }
        }

        stage('Kubernetes Verify') {

            steps {

                echo '======================================'
                echo 'STAGE 9: KUBERNETES VERIFY'
                echo '======================================'

                k8sVerify(
                    "${K8S_NAMESPACE}",
                    "${K8S_DEPLOYMENT}"
                )
            }
        }

        stage('Build Information') {

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
            echo 'FINAL CI/CD PIPELINE SUCCESS'
            echo '======================================'

            echo "Application: ${IMAGE_NAME}"

            echo "Docker Image: ${IMAGE}:${params.APP_VERSION}"

            echo "Environment: ${params.ENVIRONMENT}"

            echo "Kubernetes Namespace: ${K8S_NAMESPACE}"

            echo "Kubernetes Deployment: ${K8S_DEPLOYMENT}"
        }

        failure {

            echo '======================================'
            echo 'FINAL CI/CD PIPELINE FAILED'
            echo '======================================'

            echo 'Check the failed stage and Console Output.'
        }

        always {

            echo '======================================'
            echo 'FINAL PIPELINE FINISHED'
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

Do not put passwords or tokens in this file.

---

# Part 18 — Check the Final Jenkinsfile

Run:

```bash
grep -n "@Library" Jenkinsfile
```

Expected:

```text
@Library('jenkins-shared-library') _
```

Check agent:

```bash
grep -n "docker-agent" Jenkinsfile
```

Check shared functions:

```bash
grep -nE "buildApp|testApp|dockerBuild|dockerTest|dockerPush|k8sDeploy|k8sVerify" Jenkinsfile
```

---

# Part 19 — Check for Exposed Secrets

Run:

```bash
grep -Rni "dckr_pat" . --exclude-dir=.git
```

Expected:

```text
```

Check GitHub token patterns:

```bash
grep -Rni "ghp_" . --exclude-dir=.git
```

Expected:

```text
```

Check common secret assignments:

```bash
grep -RniE "password\s*=|token\s*=|secret\s*=" . --exclude-dir=.git
```

Review the results carefully.

There must be no actual passwords or tokens.

---

# Part 20 — Test the Application

Run:

```bash
chmod +x app.sh
```

Run:

```bash
./app.sh
```

---

# Part 21 — Test Docker Locally

Build:

```bash
docker build -t jenkins-demo-app:final-test .
```

Run:

```bash
docker run --rm jenkins-demo-app:final-test
```

Remove the test image:

```bash
docker rmi jenkins-demo-app:final-test
```

---

# Part 22 — Check Kubernetes Before Deployment

Run:

```bash
kubectl get deployment -n jenkins-demo
```

Check Pods:

```bash
kubectl get pods -n jenkins-demo
```

Check Service:

```bash
kubectl get service -n jenkins-demo
```

---

# Part 23 — Commit the Final Jenkinsfile

Run:

```bash
git status
```

Add:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add final production style Jenkins CI CD pipeline"
```

---

# Part 24 — Push the Final Jenkinsfile

Run:

```bash
git push origin main
```

---

# Part 25 — Open Jenkins

Open:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

Make sure the job uses:

```text
Pipeline script from SCM
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

# Part 26 — Run the Final Pipeline

Click:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
100.0
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

# Part 27 — Watch the Final Pipeline

The pipeline should run:

```text
Checkout
    |
    v
Validate
    |
    v
Build
    |
    v
Test
    |
    v
Docker Build
    |
    v
Docker Test
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

# Part 28 — Check the Jenkins Agent

Open:

```text
Console Output
```

Look for:

```text
docker-agent
```

You should see the pipeline executing on the Docker agent.

---

# Part 29 — Check Shared Library

Look for:

```text
SHARED LIBRARY: BUILD
```

and:

```text
SHARED LIBRARY: TEST
```

and:

```text
SHARED LIBRARY: DOCKER BUILD
```

and:

```text
SHARED LIBRARY: DOCKER TEST
```

and:

```text
SHARED LIBRARY: DOCKER PUSH
```

and:

```text
SHARED LIBRARY: KUBERNETES DEPLOY
```

and:

```text
SHARED LIBRARY: KUBERNETES VERIFY
```

---

# Part 30 — Check Docker Hub

Open:

```text
Docker Hub
```

Open:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

You should see:

```text
100.0
```

---

# Part 31 — Verify Docker Image Locally

Run:

```bash
docker images | grep jenkins-demo-app
```

You should see:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app   100.0
```

---

# Part 32 — Run Docker Image

Run:

```bash
docker run --rm YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:100.0
```

The application should run successfully.

---

# Part 33 — Verify Kubernetes Image

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:100.0
```

---

# Part 34 — Verify Kubernetes Rollout

Run:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

Expected:

```text
deployment "jenkins-demo-app" successfully rolled out
```

---

# Part 35 — Check Pods

Run:

```bash
kubectl get pods -n jenkins-demo -o wide
```

All application Pods should eventually show:

```text
Running
```

---

# Part 36 — Check Deployment

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo
```

Expected:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
jenkins-demo-app   2/2     2            2
```

The numbers may differ if you changed the replica count.

---

# Part 37 — Check Service

Run:

```bash
kubectl get service -n jenkins-demo
```

Expected:

```text
jenkins-demo-service
```

---

# Part 38 — Check All Kubernetes Resources

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

# Part 39 — Check Jenkins Build Artifact

Open the latest Jenkins build.

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

Expected:

```text
Application=jenkins-demo-app
DockerImage=YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:100.0
Environment=dev
JenkinsBuild=<build-number>
GitCommit=<commit-hash>
GitBranch=main
KubernetesNamespace=jenkins-demo
KubernetesDeployment=jenkins-demo-app
```

---

# Part 40 — Test Another Production Deployment

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
101.0
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

# Part 41 — Verify Version 101.0

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:101.0
```

---

# Part 42 — Verify Docker Hub 101.0

Open:

```text
Docker Hub
→ jenkins-demo-app
```

Verify:

```text
101.0
```

---

# Part 43 — Test Automatic GitHub Trigger

Make a small application change.

Open:

```bash
code README.md
```

Add:

```text
Final Jenkins Zero to Hero CI/CD project completed.
```

Save.

---

# Part 44 — Check Git Status

Run:

```bash
git status
```

Expected:

```text
modified: README.md
```

---

# Part 45 — Commit the Change

Run:

```bash
git add README.md
```

Commit:

```bash
git commit -m "Test final Jenkins CI CD webhook"
```

---

# Part 46 — Push to GitHub

Run:

```bash
git push origin main
```

The expected flow is:

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
Jenkins Agent
    |
    v
Shared Library
    |
    v
Docker
    |
    v
Docker Hub
    |
    v
Kubernetes
```

---

# Part 47 — Check Jenkins Automatically

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

A new build should appear if the GitHub webhook is configured and your tunnel is running.

If the webhook is not running, manually use:

```text
Build with Parameters
```

---

# Part 48 — Check Latest Git Commit

Open:

```text
Console Output
```

Look for:

```text
git log --oneline -1
```

You should see:

```text
Test final Jenkins CI CD webhook
```

---

# Part 49 — Verify Final Pipeline

The final build should contain:

```text
Checkout
Validate
Build
Test
Docker Build
Docker Test
Docker Push
Kubernetes Deploy
Kubernetes Verify
Build Information
Archive
```

---

# Part 50 — Test Failure Handling

We will intentionally break a validation command.

Open:

```bash
code Jenkinsfile
```

Inside:

```text
Validate
```

add temporarily:

```groovy
sh 'test -f final-file-that-does-not-exist.txt'
```

Save.

---

# Part 51 — Commit the Failure Test

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Test final CI CD failure handling"
```

Push:

```bash
git push origin main
```

---

# Part 52 — Run the Failed Pipeline

Open Jenkins.

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
102.0
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

The pipeline should fail during:

```text
Validate
```

---

# Part 53 — Verify Docker Image Was Not Pushed

Because validation failed before Docker Build, there should not be a new valid:

```text
102.0
```

image pushed by this pipeline.

Check Docker Hub.

The image:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:102.0
```

should not have been created by the failed pipeline.

---

# Part 54 — Verify Kubernetes Was Not Deployed

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

It should still show the previous successful image.

For example:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:101.0
```

---

# Part 55 — Fix the Pipeline

Open:

```bash
code Jenkinsfile
```

Remove:

```groovy
sh 'test -f final-file-that-does-not-exist.txt'
```

Save.

---

# Part 56 — Commit the Fix

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Fix final CI CD validation"
```

Push:

```bash
git push origin main
```

---

# Part 57 — Run Successful Pipeline Again

Open Jenkins.

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
103.0
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

# Part 58 — Verify Version 103.0

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:103.0
```

---

# Part 59 — Verify Final Pods

Run:

```bash
kubectl get pods -n jenkins-demo -o wide
```

Pods should show:

```text
Running
```

---

# Part 60 — Verify Final Rollout

Run:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

Expected:

```text
deployment "jenkins-demo-app" successfully rolled out
```

---

# Part 61 — Check Kubernetes Rollout History

Run:

```bash
kubectl rollout history deployment/jenkins-demo-app -n jenkins-demo
```

You should see multiple revisions.

---

# Part 62 — Test Kubernetes Rollback

Check the current image:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Rollback:

```bash
kubectl rollout undo deployment/jenkins-demo-app -n jenkins-demo
```

Wait:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

Check:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

You should now have the previous revision.

---

# Part 63 — Restore the Desired Version Through Jenkins

Run the pipeline again.

Use:

```text
APP_VERSION:
104.0
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

Verify:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:104.0
```

---

# Part 64 — Final System Verification

Check Docker:

```bash
docker ps
```

Check Jenkins:

```bash
docker ps --filter "name=jenkins"
```

Check Jenkins Docker CLI:

```bash
docker exec jenkins docker --version
```

Check Jenkins kubectl:

```bash
docker exec jenkins kubectl version --client
```

Check Kubernetes nodes:

```bash
kubectl get nodes
```

Check namespace:

```bash
kubectl get namespace jenkins-demo
```

Check deployment:

```bash
kubectl get deployment -n jenkins-demo
```

Check Pods:

```bash
kubectl get pods -n jenkins-demo
```

Check Service:

```bash
kubectl get service -n jenkins-demo
```

---

# Part 65 — Final Docker Verification

Run:

```bash
docker images | grep jenkins-demo-app
```

You should have several versions from your labs.

---

# Part 66 — Final Docker Hub Verification

Open:

```text
Docker Hub
```

Open:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

You should see the image tags created during your labs.

---

# Part 67 — Final Jenkins Verification

Open:

```text
http://localhost:8081
```

Verify:

```text
lab-05-pipeline
```

Check:

```text
Build History
```

You should have:

```text
Successful Builds
Failed Builds
Successful Builds
```

This proves you tested both failure and recovery.

---

# Part 68 — Final Shared Library Verification

Open:

```text
GitHub
→ jenkins-shared-library
```

Verify:

```text
vars/buildApp.groovy
vars/testApp.groovy
vars/dockerBuild.groovy
vars/dockerTest.groovy
vars/dockerPush.groovy
vars/k8sDeploy.groovy
vars/k8sVerify.groovy
```

---

# Part 69 — Final Credentials Verification

Open:

```text
Jenkins
→ Manage Jenkins
→ Credentials
→ Global
```

Verify:

```text
dockerhub-credentials
```

Make sure no token or password is stored in:

```text
Jenkinsfile
GitHub
Dockerfile
README
```

---

# Part 70 — Final Security Verification

Open:

```text
Manage Jenkins
→ Security
```

Verify:

```text
Authentication enabled
Authorization enabled
User signup disabled
CSRF protection enabled
```

Verify that:

```text
developer
```

does not have:

```text
Overall → Administer
```

Verify:

```text
viewer
```

does not have:

```text
Job → Build
```

---

# Part 71 — Final Git Status

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

# Part 72 — Final Git Log

Run:

```bash
git log --oneline --max-count=15
```

You should see commits related to:

```text
CI/CD
Docker
Docker Hub
Kubernetes
Shared Library
Webhook
Security
```

---

# Part 73 — Final Shared Library Git Status

Run:

```bash
cd /c/project/jenkins-shared-library
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

# Part 74 — Save Lab 16 Documentation

Go to the Jenkins documentation repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go to:

```bash
cd labs
```

Create:

```bash
touch lab-16-final-cicd-project.md
```

Open:

```bash
code lab-16-final-cicd-project.md
```

Paste this complete lab into the file.

Save.

---

# Part 75 — Check Git Status

Return to repository root:

```bash
cd /c/project/jenkins-zero-to-hero
```

Run:

```bash
git status
```

You should see:

```text
new file: labs/lab-16-final-cicd-project.md
```

---

# Part 76 — Add Lab 16

```bash
git add labs/lab-16-final-cicd-project.md
```

Check:

```bash
git status
```

---

# Part 77 — Commit Lab 16

```bash
git commit -m "Add Jenkins Lab 16 final CI CD project"
```

---

# Part 78 — Push Lab 16

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

# Part 79 — Final Jenkins Repository Structure

Your final Jenkins documentation repository should now look like:

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
    ├── lab-11-complete-cicd-pipeline.md
    │
    ├── lab-12-jenkins-credentials.md
    │
    ├── lab-13-jenkins-agents.md
    │
    ├── lab-14-jenkins-shared-libraries.md
    │
    ├── lab-15-jenkins-security.md
    │
    └── lab-16-final-cicd-project.md
```

---

# Lab 16 Completion Checklist

```text
[ ] Jenkins is running
[ ] Docker is running
[ ] Minikube is running
[ ] Kubernetes nodes are Ready
[ ] Jenkins Docker access verified
[ ] Jenkins kubectl access verified
[ ] Docker Hub repository verified
[ ] Jenkins Docker Hub credentials verified
[ ] Shared Library verified
[ ] Docker agent verified
[ ] Application repository verified
[ ] Final Jenkinsfile created
[ ] Shared Library imported
[ ] Docker agent selected
[ ] Checkout stage completed
[ ] Validate stage completed
[ ] Build stage completed
[ ] Test stage completed
[ ] Docker Build completed
[ ] Docker Test completed
[ ] Docker Login completed
[ ] Docker Push completed
[ ] Docker Hub image verified
[ ] Kubernetes Deploy completed
[ ] Kubernetes Verify completed
[ ] Kubernetes Pods verified
[ ] Build information created
[ ] Artifact archived
[ ] GitHub webhook tested
[ ] Automatic Jenkins build tested
[ ] Shared Library functions tested
[ ] Jenkins agent tested
[ ] Jenkins credentials tested
[ ] Jenkins security reviewed
[ ] Pipeline failure tested
[ ] Pipeline failure fixed
[ ] Kubernetes rollback tested
[ ] Final deployment verified
[ ] Git status clean
[ ] Lab 16 documentation saved
[ ] Lab 16 committed
[ ] Lab 16 pushed to GitHub
```

---

# Final Cleanup

## Option 1 — Keep Everything

Recommended if you want to keep this as a portfolio project.

Keep:

```text
jenkins-zero-to-hero
jenkins-demo-app
jenkins-shared-library
Jenkins
Docker Hub repository
Minikube
Kubernetes
```

This gives you a complete DevOps project that you can continue improving.

---

# Option 2 — Delete Kubernetes Application

Delete Deployment:

```bash
kubectl delete deployment jenkins-demo-app -n jenkins-demo
```

Delete Service:

```bash
kubectl delete service jenkins-demo-service -n jenkins-demo
```

Check:

```bash
kubectl get all -n jenkins-demo
```

---

# Option 3 — Delete Kubernetes Namespace

To delete everything in the namespace:

```bash
kubectl delete namespace jenkins-demo
```

Verify:

```bash
kubectl get namespaces
```

---

# Option 4 — Remove Local Docker Images

Check:

```bash
docker images | grep jenkins-demo-app
```

Remove a specific image:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:100.0
```

Remove another:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:101.0
```

Remove another:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:103.0
```

Remove another:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:104.0
```

Only remove images you no longer need.

---

# Option 5 — Remove Dangling Images

Check:

```bash
docker images --filter "dangling=true"
```

Remove:

```bash
docker image prune
```

---

# Option 6 — Stop Minikube

If finished with Kubernetes:

```bash
minikube stop
```

Check:

```bash
minikube status
```

---

# Option 7 — Stop Jenkins

If finished with Jenkins:

```bash
docker stop jenkins
```

Check:

```bash
docker ps
```

---

# Option 8 — Complete Jenkins Removal

Only use this if you want to permanently delete Jenkins and its data.

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

Check:

```bash
docker volume ls
```

Check:

```bash
docker network ls
```

---

# Final Project Summary

You have now built a complete Jenkins DevOps workflow:

```text
                         Developer
                             |
                             | git push
                             v
                          GitHub
                             |
                             | webhook
                             v
                    Jenkins Controller
                             |
                             v
                       Jenkins Agent
                             |
                             v
                    Jenkins Shared Library
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
           Validate         Test       Docker Build
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
                                           Pods
                                             |
                                             v
                                        Application
```

---

# Jenkins Zero to Hero — Final Skill Map

```text
Jenkins Installation
        |
        v
Jenkins Jobs
        |
        v
Git + GitHub
        |
        v
GitHub Webhooks
        |
        v
Jenkins Pipeline
        |
        v
Jenkinsfile
        |
        v
CI Pipeline
        |
        v
Docker
        |
        v
Docker Hub
        |
        v
Kubernetes
        |
        v
Complete CI/CD
        |
        v
Credentials
        |
        v
Agents
        |
        v
Shared Libraries
        |
        v
Security
        |
        v
Production-Style CI/CD
```

---

# Final Portfolio Project

Your final project can be described as:

```text
Production-Style Jenkins CI/CD Pipeline

GitHub → Jenkins → Docker → Docker Hub → Kubernetes
```

Technologies:

```text
Git
GitHub
Jenkins
Jenkinsfile
Jenkins Shared Libraries
Jenkins Agents
Docker
Docker Hub
Kubernetes
Minikube
kubectl
CI/CD
Webhooks
Jenkins Credentials
RBAC / Security Concepts
```

---

# What You Can Now Explain in an Interview

You should now be able to explain:

```text
What is Jenkins?

What is a Jenkins Job?

What is a Jenkins Pipeline?

What is a Jenkinsfile?

What is Declarative Pipeline?

What is a Jenkins Controller?

What is a Jenkins Agent?

What is an Executor?

Why do we use Jenkins Agents?

What is a Jenkins Shared Library?

Why should secrets not be stored in Jenkinsfile?

How does Jenkins access Docker?

How does Jenkins build a Docker image?

How does Jenkins push an image to Docker Hub?

How does Jenkins deploy to Kubernetes?

What is a Kubernetes Deployment?

What is a Kubernetes Service?

What is a rolling update?

How do you rollback a Kubernetes Deployment?

What is a GitHub webhook?

How does Git push trigger Jenkins?

How do Jenkins Credentials work?

Why is Docker socket access sensitive?

Why should Jenkins use least privilege?

How do you troubleshoot a failed pipeline?
```

---

# Final CI/CD Flow to Remember

```text
git push
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
Agent
   |
   v
Checkout
   |
   v
Validate
   |
   v
Test
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
Application
```

# End of Jenkins Zero to Hero
