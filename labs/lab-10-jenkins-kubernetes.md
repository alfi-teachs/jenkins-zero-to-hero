# Jenkins Zero to Hero — Lab 10

## Jenkins + Kubernetes

This lab continues from:

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
```

In Lab 9, Jenkins built a Docker image and pushed it to Docker Hub.

In this lab, Jenkins will deploy that Docker image to Kubernetes.

The workflow becomes:

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
    +---- Test
    |
    +---- Docker Build
    |
    +---- Docker Push
    |
    v
Docker Hub
    |
    | Kubernetes pulls image
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

# Lab Objective

By the end of this lab, you will understand:

```text
Jenkins + Kubernetes
kubectl
Kubernetes Deployment
Kubernetes Service
Jenkins Kubernetes Deployment
Docker Hub Image
Kubernetes Namespace
Image Tags
Rolling Deployment
Deployment Verification
Jenkins Deployment Pipeline
```

---

# Important

This lab uses:

```text
Jenkins
Docker
Docker Hub
Minikube
Kubernetes
kubectl
```

Your application image will come from Docker Hub.

Example:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

Kubernetes will pull that image and create the application Pods.

---

# Prerequisites

Docker Desktop must be running.

Check:

```bash
docker ps
```

Jenkins must be running:

```bash
docker ps --filter "name=jenkins"
```

Check Docker CLI inside Jenkins:

```bash
docker exec jenkins docker --version
```

Check Minikube:

```bash
minikube version
```

Check kubectl:

```bash
kubectl version --client
```

---

# Part 1 — Check Minikube

Run:

```bash
minikube status
```

You should see something similar to:

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

Expected:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   ...
```

If you are using a multi-node Minikube profile, you may see:

```text
minikube
minikube-m02
minikube-m03
```

All nodes should be:

```text
Ready
```

---

# Part 3 — Check Kubernetes Version

Run:

```bash
kubectl version
```

You should get Kubernetes client and server information.

---

# Part 4 — Check Current Kubernetes Context

Run:

```bash
kubectl config current-context
```

Expected:

```text
minikube
```

If you use another Minikube profile, your context may be different.

---

# Part 5 — Check Minikube IP

Run:

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

Your IP may be different.

---

# Part 6 — Check Kubernetes Namespaces

Run:

```bash
kubectl get namespaces
```

You should see namespaces similar to:

```text
default
kube-node-lease
kube-public
kube-system
```

---

# Part 7 — Create a Jenkins Kubernetes Namespace

We will deploy the application into its own namespace.

Run:

```bash
kubectl create namespace jenkins-demo
```

Check:

```bash
kubectl get namespaces
```

You should see:

```text
jenkins-demo
```

If the namespace already exists, that is okay.

---

# Part 8 — Create the Kubernetes Deployment File Locally

Go to the application repository:

```bash
cd /c/project/jenkins-demo-app
```

Create:

```bash
touch k8s-deployment.yaml
```

Create the Kubernetes Service file:

```bash
touch k8s-service.yaml
```

Check:

```bash
ls
```

You should see:

```text
Dockerfile
Jenkinsfile
README.md
app.sh
version.txt
k8s-deployment.yaml
k8s-service.yaml
```

---

# Part 9 — Create Kubernetes Deployment

Open:

```bash
code k8s-deployment.yaml
```

Paste:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: jenkins-demo-app
  namespace: jenkins-demo

spec:

  replicas: 2

  selector:
    matchLabels:
      app: jenkins-demo-app

  template:

    metadata:
      labels:
        app: jenkins-demo-app

    spec:

      containers:

        - name: jenkins-demo-app

          image: YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0

          imagePullPolicy: Always

          ports:
            - containerPort: 8080

          resources:
            requests:
              cpu: "50m"
              memory: "32Mi"

            limits:
              cpu: "200m"
              memory: "128Mi"
```

Replace:

```text
YOUR-DOCKERHUB-USERNAME
```

with your Docker Hub username.

For example:

```yaml
image: alfiademo/jenkins-demo-app:10.0
```

Your image tag must actually exist in Docker Hub.

---

# Part 10 — Understand the Deployment

The Deployment creates:

```text
2 replicas
```

That means Kubernetes will try to maintain:

```text
2 Pods
```

The Pods will have the label:

```text
app: jenkins-demo-app
```

The image comes from:

```text
Docker Hub
```

---

# Part 11 — Create Kubernetes Service

Open:

```bash
code k8s-service.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: jenkins-demo-service
  namespace: jenkins-demo

spec:

  type: NodePort

  selector:
    app: jenkins-demo-app

  ports:

    - protocol: TCP

      port: 80

      targetPort: 8080

      nodePort: 30080
```

Save.

---

# Important Note About This Application

Our current `app.sh` application is a shell script.

It prints output and exits.

It does not actually listen on:

```text
8080
```

Therefore, Kubernetes can create the Pods, but this exact application is not a real HTTP server.

That is intentional for this first Kubernetes integration lab.

The main objective is:

```text
Jenkins
    |
    v
Docker Hub
    |
    v
Kubernetes
    |
    v
Deployment
    |
    v
Pods
```

In a later lab, we will use a real web application and expose it through a Kubernetes Service.

---

# Part 12 — Validate the Kubernetes YAML

Run:

```bash
kubectl apply --dry-run=client -f k8s-deployment.yaml
```

Expected:

```text
deployment.apps/jenkins-demo-app created
```

or an equivalent dry-run message.

Validate the Service:

```bash
kubectl apply --dry-run=client -f k8s-service.yaml
```

---

# Part 13 — Deploy Manually First

Before Jenkins deploys the application, we should make sure the Kubernetes manifests work.

Run:

```bash
kubectl apply -f k8s-deployment.yaml
```

Expected:

```text
deployment.apps/jenkins-demo-app created
```

Apply the Service:

```bash
kubectl apply -f k8s-service.yaml
```

Expected:

```text
service/jenkins-demo-service created
```

---

# Part 14 — Check Deployment

Run:

```bash
kubectl get deployments -n jenkins-demo
```

Expected:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
jenkins-demo-app   2/2     2            2
```

---

# Part 15 — Check Pods

Run:

```bash
kubectl get pods -n jenkins-demo
```

You should see two Pods.

Example:

```text
NAME                                READY   STATUS
jenkins-demo-app-xxxxxxxxxx-xxxxx   1/1     Running
jenkins-demo-app-xxxxxxxxxx-yyyyy   1/1     Running
```

Your Pod names will be different.

---

# Part 16 — Check Pod Details

Run:

```bash
kubectl get pods -n jenkins-demo -o wide
```

You will see:

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

# Part 17 — Check Service

Run:

```bash
kubectl get service -n jenkins-demo
```

Expected:

```text
NAME                  TYPE       CLUSTER-IP    PORT(S)
jenkins-demo-service  NodePort   ...           80:30080/TCP
```

---

# Part 18 — Check Deployment Details

Run:

```bash
kubectl describe deployment jenkins-demo-app -n jenkins-demo
```

Look for:

```text
Replicas:
2 desired
2 updated
2 total
2 available
```

---

# Part 19 — Check Pod Logs

Get Pod names:

```bash
kubectl get pods -n jenkins-demo
```

Copy one Pod name.

Then:

```bash
kubectl logs -n jenkins-demo POD-NAME
```

Replace:

```text
POD-NAME
```

with the actual Pod name.

For example:

```bash
kubectl logs -n jenkins-demo jenkins-demo-app-xxxxxxxxxx-xxxxx
```

You should see the application output.

---

# Part 20 — Check Image Used by the Pod

Run:

```bash
kubectl get pods -n jenkins-demo -o jsonpath="{.items[*].spec.containers[*].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

---

# Part 21 — Check Deployment Image

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

---

# Part 22 — Understand the Kubernetes Flow

The manual deployment is:

```text
Docker Hub
    |
    | Pull Image
    v
Kubernetes
    |
    v
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
```

---

# Part 23 — Create Kubernetes Namespace YAML

Instead of manually creating the namespace, we will put it into source control.

Create:

```bash
touch k8s-namespace.yaml
```

Open:

```bash
code k8s-namespace.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: jenkins-demo
```

Save.

---

# Part 24 — Check Namespace YAML

Run:

```bash
cat k8s-namespace.yaml
```

Expected:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: jenkins-demo
```

---

# Part 25 — Update Deployment YAML

The deployment currently has:

```text
10.0
```

Later Jenkins will replace this tag automatically.

Keep the current version for now.

---

# Part 26 — Verify All Kubernetes Files

Run:

```bash
ls k8s-*.yaml
```

Expected:

```text
k8s-deployment.yaml
k8s-namespace.yaml
k8s-service.yaml
```

---

# Part 27 — Understand Jenkins Kubernetes Access

Jenkins needs:

```text
kubectl
```

and Kubernetes credentials/configuration.

The Jenkins container currently has Docker support, but it does not automatically have access to your Minikube cluster.

We will add:

```text
kubectl
+
Kubernetes kubeconfig
```

to the Jenkins container.

---

# Part 28 — Check Your Local Kubernetes Configuration

Run:

```bash
kubectl config view --minify
```

You should see the current Minikube cluster configuration.

Check only the server address:

```bash
kubectl config view --minify -o jsonpath="{.clusters[0].cluster.server}"
```

You may see something similar to:

```text
https://127.0.0.1:xxxxx
```

The port will be different on your machine.

---

# Part 29 — Check the Minikube API Server Address

Run:

```bash
minikube status
```

Then:

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

Your value may be different.

---

# Part 30 — Important Kubernetes Access Concept

From Windows:

```text
kubectl
   |
   v
Minikube
```

But Jenkins is inside:

```text
Docker Container
```

So:

```text
Jenkins Container
   |
   X
localhost
```

inside the Jenkins container is not the same as:

```text
Windows localhost
```

Therefore, we need to make Kubernetes reachable from the Jenkins container.

For this lab, we will configure Jenkins to use the host gateway:

```text
host.docker.internal
```

---

# Part 31 — Prepare a Jenkins Kubernetes Directory

Go to:

```bash
cd /c/project/jenkins-docker
```

Create:

```bash
mkdir kubernetes
```

Enter:

```bash
cd kubernetes
```

---

# Part 32 — Check kubectl on Windows

Run:

```bash
kubectl version --client
```

Make sure the command works before continuing.

---

# Part 33 — Create a Temporary Jenkins Kubernetes Config

Go back to the project directory:

```bash
cd /c/project/jenkins-docker
```

Create:

```bash
touch kubeconfig-jenkins
```

Open:

```bash
code kubeconfig-jenkins
```

Paste the kubeconfig content from:

```bash
kubectl config view --raw
```

Run this command to display it:

```bash
kubectl config view --raw
```

Copy the complete output into:

```text
kubeconfig-jenkins
```

Save it.

---

# Part 34 — Important Kubeconfig Adjustment

Inside:

```text
kubeconfig-jenkins
```

look for:

```text
server: https://127.0.0.1:xxxxx
```

Change:

```text
127.0.0.1
```

to:

```text
host.docker.internal
```

For example:

```text
server: https://host.docker.internal:32769
```

Use the same port that appeared in your original kubeconfig.

---

# Part 35 — Lab-Only TLS Note

Because the Kubernetes API certificate may have been created for:

```text
127.0.0.1
```

you may encounter a certificate hostname error when using:

```text
host.docker.internal
```

For this learning lab only, the kubeconfig cluster entry can use:

```yaml
insecure-skip-tls-verify: true
```

and remove:

```yaml
certificate-authority-data:
```

from that cluster entry.

Do not use this approach as your production Kubernetes security model.

---

# Part 36 — Install kubectl in the Jenkins Image

Go to:

```bash
cd /c/project/jenkins-docker
```

Open:

```bash
code Dockerfile
```

Replace the Dockerfile with:

```dockerfile
FROM jenkins/jenkins:lts

USER root

RUN apt-get update \
    && apt-get install -y ca-certificates curl \
    && install -m 0755 -d /etc/apt/keyrings \
    && curl -fsSL https://download.docker.com/linux/debian/gpg \
       -o /etc/apt/keyrings/docker.asc \
    && chmod a+r /etc/apt/keyrings/docker.asc \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
       | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt-get update \
    && apt-get install -y docker-ce-cli \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl \
    && rm kubectl

USER jenkins
```

Save.

---

# Part 37 — Build the Updated Jenkins Image

Run:

```bash
docker build -t jenkins-docker:lts .
```

Check:

```bash
docker images | grep jenkins-docker
```

---

# Part 38 — Stop Jenkins

Run:

```bash
docker stop jenkins
```

Remove the current container:

```bash
docker rm jenkins
```

Do not delete:

```text
jenkins_home
```

---

# Part 39 — Start Jenkins with kubectl

We will mount the kubeconfig file into Jenkins.

Run:

```bash
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  --network jenkins \
  -p 8081:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /c/project/jenkins-docker/kubeconfig-jenkins:/var/jenkins_home/.kube/config:ro \
  jenkins-docker:lts
```

---

# Part 40 — Check Jenkins

Run:

```bash
docker ps
```

Expected:

```text
jenkins
```

---

# Part 41 — Check kubectl Inside Jenkins

Run:

```bash
docker exec jenkins kubectl version --client
```

You should see the kubectl client version.

---

# Part 42 — Check Kubernetes Nodes from Jenkins

Run:

```bash
docker exec jenkins kubectl get nodes
```

The goal is to see the same Kubernetes nodes that your Windows terminal sees.

Expected format:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   ...
```

If this command works, Jenkins can communicate with Kubernetes.

---

# Part 43 — Check Jenkins Kubernetes Context

Run:

```bash
docker exec jenkins kubectl config current-context
```

Expected:

```text
minikube
```

---

# Part 44 — Check Kubernetes Namespaces from Jenkins

Run:

```bash
docker exec jenkins kubectl get namespaces
```

You should see:

```text
jenkins-demo
```

---

# Part 45 — Check Existing Application from Jenkins

Run:

```bash
docker exec jenkins kubectl get deployments -n jenkins-demo
```

Expected:

```text
jenkins-demo-app
```

Check Pods:

```bash
docker exec jenkins kubectl get pods -n jenkins-demo
```

---

# Part 46 — Test kubectl from Jenkins Manually

Run:

```bash
docker exec jenkins kubectl get pods -n jenkins-demo -o wide
```

This confirms that Jenkins can query your Minikube cluster.

---

# Part 47 — Create the Jenkins Kubernetes Deployment Stage

Go to the application repository:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

We will add Kubernetes deployment to the existing pipeline.

Add these environment variables:

```groovy
K8S_NAMESPACE = 'jenkins-demo'
K8S_DEPLOYMENT = 'jenkins-demo-app'
```

Your environment section should look like:

```groovy
environment {
    DOCKERHUB_USERNAME = 'YOUR-DOCKERHUB-USERNAME'
    IMAGE_NAME = 'jenkins-demo-app'
    IMAGE = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}"
    K8S_NAMESPACE = 'jenkins-demo'
    K8S_DEPLOYMENT = 'jenkins-demo-app'
}
```

---

# Part 48 — Add Kubernetes Deployment Stage

Add this stage after the Docker Push stage:

```groovy
stage('Kubernetes Deploy') {
    steps {

        echo '======================================'
        echo 'KUBERNETES DEPLOY'
        echo '======================================'

        sh 'kubectl get nodes'

        sh 'kubectl get namespace "$K8S_NAMESPACE"'

        sh 'kubectl set image deployment/"$K8S_DEPLOYMENT" "$IMAGE_NAME"="$IMAGE:$APP_VERSION" -n "$K8S_NAMESPACE"'

        sh 'kubectl rollout status deployment/"$K8S_DEPLOYMENT" -n "$K8S_NAMESPACE" --timeout=120s'
    }
}
```

Save.

---

# Part 49 — Understand the Kubernetes Deployment Command

This command:

```bash
kubectl set image deployment/"$K8S_DEPLOYMENT" "$IMAGE_NAME"="$IMAGE:$APP_VERSION" -n "$K8S_NAMESPACE"
```

changes the image used by the existing Deployment.

For example:

```text
Current:

jenkins-demo-app
    image:
    user/jenkins-demo-app:10.0
```

After the Jenkins build:

```text
jenkins-demo-app
    image:
    user/jenkins-demo-app:12.0
```

---

# Part 50 — Understand Rollout Status

This command:

```bash
kubectl rollout status deployment/"$K8S_DEPLOYMENT" -n "$K8S_NAMESPACE" --timeout=120s
```

waits for the Deployment to finish updating.

If the rollout succeeds:

```text
deployment successfully rolled out
```

If Kubernetes cannot update the Pods:

```text
Jenkins build fails
```

That is exactly what we want in a CI/CD pipeline.

---

# Part 51 — Add Kubernetes Verification Stage

After the deployment stage, add:

```groovy
stage('Kubernetes Verify') {
    steps {

        echo '======================================'
        echo 'KUBERNETES VERIFY'
        echo '======================================'

        sh 'kubectl get deployment "$K8S_DEPLOYMENT" -n "$K8S_NAMESPACE"'

        sh 'kubectl get pods -n "$K8S_NAMESPACE" -l app="$K8S_DEPLOYMENT" -o wide'

        sh 'kubectl get service -n "$K8S_NAMESPACE"'

        sh 'kubectl get deployment "$K8S_DEPLOYMENT" -n "$K8S_NAMESPACE" -o jsonpath="{.spec.template.spec.containers[0].image}"'
    }
}
```

Save.

---

# Part 52 — Check the Jenkinsfile

Run:

```bash
grep -n "Kubernetes" Jenkinsfile
```

You should see:

```text
Kubernetes Deploy
Kubernetes Verify
```

Check the full file:

```bash
cat Jenkinsfile
```

---

# Part 53 — Commit the Kubernetes Pipeline

Run:

```bash
git status
```

You should see:

```text
modified: Jenkinsfile
```

Add:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add Jenkins Kubernetes deployment"
```

Push:

```bash
git push origin main
```

---

# Part 54 — Build the Docker Image

Open Jenkins:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

Click:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
13.0
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

# Part 55 — Understand the New Pipeline

The pipeline now becomes:

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
Archive
```

---

# Part 56 — Check Docker Hub

Open Docker Hub.

You should see:

```text
13.0
```

under:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

---

# Part 57 — Check Kubernetes Deployment

From your Windows Git Bash:

```bash
kubectl get deployment -n jenkins-demo
```

You should see:

```text
jenkins-demo-app
```

---

# Part 58 — Check Kubernetes Pods

Run:

```bash
kubectl get pods -n jenkins-demo
```

You should see:

```text
2 Pods
```

They should eventually become:

```text
Running
```

---

# Part 59 — Check the Pod Image

Run:

```bash
kubectl get pods -n jenkins-demo -o jsonpath="{.items[*].spec.containers[*].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:13.0
```

---

# Part 60 — Check the Deployment Image

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:13.0
```

---

# Part 61 — Check Rollout

Run:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

Expected:

```text
deployment "jenkins-demo-app" successfully rolled out
```

---

# Part 62 — Check ReplicaSets

Run:

```bash
kubectl get replicasets -n jenkins-demo
```

You may see:

```text
jenkins-demo-app-xxxxxxxx
```

The new ReplicaSet corresponds to the updated Deployment version.

---

# Part 63 — Check Pods Across Nodes

Run:

```bash
kubectl get pods -n jenkins-demo -o wide
```

You will see:

```text
NAME
READY
STATUS
RESTARTS
AGE
IP
NODE
```

With a multi-node Minikube cluster, Pods may be placed on different nodes.

---

# Part 64 — Check Deployment History

Run:

```bash
kubectl rollout history deployment/jenkins-demo-app -n jenkins-demo
```

You should see Deployment revision history.

---

# Part 65 — Check Deployment Details

Run:

```bash
kubectl describe deployment jenkins-demo-app -n jenkins-demo
```

Look for:

```text
Image:
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:13.0
```

---

# Part 66 — Check Kubernetes Service

Run:

```bash
kubectl get service -n jenkins-demo
```

Expected:

```text
jenkins-demo-service
```

Because the application is not a web server, we are not using the Service for application traffic yet.

The Service is included to introduce the Kubernetes deployment pattern.

---

# Part 67 — Test Another Version

Open Jenkins:

```text
http://localhost:8081
```

Click:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
14.0
```

Choose:

```text
ENVIRONMENT:
test
```

Click:

```text
Build
```

---

# Part 68 — Check Kubernetes After the New Build

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:14.0
```

---

# Part 69 — Check Rollout Again

Run:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

Expected:

```text
deployment "jenkins-demo-app" successfully rolled out
```

---

# Part 70 — Check Pods Again

Run:

```bash
kubectl get pods -n jenkins-demo -o wide
```

The Pods should be running the new Deployment revision.

---

# Part 71 — Understand Rolling Update

When the image changes:

```text
Version 13.0
       |
       v
Version 14.0
```

Kubernetes updates the Deployment.

The flow is:

```text
Old Pod
   |
   v
New Pod
   |
   v
New Pod Ready
   |
   v
Old Pod Removed
```

This is part of Kubernetes' Deployment rolling-update behavior.

---

# Part 72 — Check Deployment Revision

Run:

```bash
kubectl rollout history deployment/jenkins-demo-app -n jenkins-demo
```

You should see multiple revisions after multiple image updates.

---

# Part 73 — Test Rollback

We will intentionally roll back the Deployment.

First check history:

```bash
kubectl rollout history deployment/jenkins-demo-app -n jenkins-demo
```

Then run:

```bash
kubectl rollout undo deployment/jenkins-demo-app -n jenkins-demo
```

Check:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

---

# Part 74 — Check the Image After Rollback

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

The image should return to the previous Deployment revision.

The exact version depends on your rollout history.

---

# Part 75 — Understand Kubernetes Rollback

Without rollback:

```text
13.0
  |
  v
14.0
  |
  v
Bad release
```

With rollback:

```text
13.0
  |
  v
14.0
  |
  X
Problem
  |
  v
Rollback
  |
  v
13.0
```

---

# Part 76 — Run Jenkins Again

We want Jenkins to restore the version we want.

Run Jenkins:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
15.0
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

After the pipeline finishes:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:15.0
```

---

# Part 77 — Check Jenkins Kubernetes Access

Run:

```bash
docker exec jenkins kubectl get nodes
```

Expected:

```text
minikube   Ready
```

Run:

```bash
docker exec jenkins kubectl get pods -n jenkins-demo
```

Expected:

```text
jenkins-demo-app
```

---

# Part 78 — Check Docker Access From Jenkins

Run:

```bash
docker exec jenkins docker ps
```

Jenkins should be able to communicate with Docker.

---

# Part 79 — Jenkins Now Controls Both Docker and Kubernetes

Your Jenkins container can now communicate with:

```text
Docker
```

and:

```text
Kubernetes
```

Architecture:

```text
                 Jenkins
                    |
          +---------+---------+
          |                   |
          v                   v
       Docker             kubectl
          |                   |
          v                   v
    Docker Hub            Minikube
                              |
                              v
                        Kubernetes
                              |
                    +---------+---------+
                    |                   |
                    v                   v
                Deployment          Service
                    |
                    v
                  Pods
```

---

# Part 80 — Understand the Full CI/CD Flow

You now have:

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
    v
Deployment
    |
    v
Pods
```

---

# Part 81 — This Is Now CI/CD

Previously:

```text
CI
```

meant:

```text
GitHub
    |
    v
Jenkins
    |
    v
Build
    |
    v
Test
```

Now:

```text
CI/CD
```

means:

```text
GitHub
    |
    v
Jenkins
    |
    +---- Build
    |
    +---- Test
    |
    +---- Docker
    |
    +---- Push
    |
    +---- Deploy
    |
    v
Kubernetes
```

---

# Part 82 — Check Git Status

Go to the application repository:

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

# Part 83 — Check Git History

Run:

```bash
git log --oneline --max-count=10
```

You should see your Jenkins and Kubernetes-related commits.

---

# Part 84 — Check Kubernetes Resources

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

# Part 85 — Check All Pods

Run:

```bash
kubectl get pods -n jenkins-demo -o wide
```

All application Pods should be:

```text
Running
```

---

# Part 86 — Check Deployment

Run:

```bash
kubectl get deployment -n jenkins-demo
```

Expected:

```text
jenkins-demo-app
```

---

# Part 87 — Check Service

Run:

```bash
kubectl get service -n jenkins-demo
```

Expected:

```text
jenkins-demo-service
```

---

# Part 88 — Check the Current Image

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected format:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:<version>
```

---

# Part 89 — Save Lab 10 Documentation

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
touch lab-10-jenkins-kubernetes.md
```

Open:

```bash
code lab-10-jenkins-kubernetes.md
```

Paste this entire lab into the file.

Save.

---

# Part 90 — Check Git Status

Go back to the repository root:

```bash
cd /c/project/jenkins-zero-to-hero
```

Run:

```bash
git status
```

You should see:

```text
new file: labs/lab-10-jenkins-kubernetes.md
```

---

# Part 91 — Add Lab 10

```bash
git add labs/lab-10-jenkins-kubernetes.md
```

Check:

```bash
git status
```

---

# Part 92 — Commit Lab 10

```bash
git commit -m "Add Jenkins Lab 10 Kubernetes deployment"
```

---

# Part 93 — Push Lab 10

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

# Part 94 — Verify Jenkins Repository

Your documentation repository should now contain:

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
    └── lab-10-jenkins-kubernetes.md
```

---

# Lab 10 Completion Checklist

```text
[ ] Minikube is running
[ ] kubectl is working
[ ] Kubernetes nodes are Ready
[ ] Kubernetes namespace created
[ ] Kubernetes Deployment created
[ ] Kubernetes Service created
[ ] Docker Hub image available
[ ] Kubernetes manually pulls Docker Hub image
[ ] Kubernetes Pods created
[ ] Kubernetes Deployment verified
[ ] Kubernetes Service verified
[ ] Jenkins kubectl installed
[ ] Jenkins kubeconfig configured
[ ] Jenkins can access Kubernetes
[ ] Jenkins can run kubectl
[ ] Kubernetes Deploy stage added
[ ] Kubernetes Verify stage added
[ ] Docker image built by Jenkins
[ ] Docker image pushed by Jenkins
[ ] Kubernetes Deployment updated by Jenkins
[ ] Rollout completed
[ ] Kubernetes image verified
[ ] Second version deployed
[ ] Rollout history checked
[ ] Rollback tested
[ ] Jenkins redeployed the application
[ ] Final Pods verified
[ ] Lab 10 documentation saved
[ ] Lab 10 committed
[ ] Lab 10 pushed to GitHub
```

---

# Lab 10 Cleanup

## Recommended

If you are continuing to Lab 11, keep:

```text
Jenkins
Docker
Minikube
jenkins-demo namespace
jenkins-demo-app Deployment
jenkins-demo-service
```

We will reuse them.

---

# Delete Only the Kubernetes Application

If you want to remove the Deployment:

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

# Delete the Kubernetes Namespace

If you want to remove everything inside the namespace:

```bash
kubectl delete namespace jenkins-demo
```

Check:

```bash
kubectl get namespaces
```

---

# Recreate the Namespace Later

If you deleted it and need it again:

```bash
kubectl create namespace jenkins-demo
```

---

# Remove Docker Images

List:

```bash
docker images | grep jenkins-demo-app
```

Remove a specific image:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:13.0
```

Remove another:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:14.0
```

Only remove versions you no longer need.

---

# Remove Unused Docker Images

Check:

```bash
docker images --filter "dangling=true"
```

Remove dangling images:

```bash
docker image prune
```

---

# Stop Minikube

Only do this when you are finished with the Kubernetes labs for the day.

```bash
minikube stop
```

Check:

```bash
minikube status
```

---

# Start Minikube Again

When continuing the labs:

```bash
minikube start
```

Check:

```bash
minikube status
```

---

# Complete Jenkins Cleanup

Only use this if you want to completely remove Jenkins.

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

Remove Jenkins network:

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

# Important Security Note

The Jenkins container currently has access to:

```text
Docker socket
```

and:

```text
Kubernetes credentials
```

That gives Jenkins significant control over your local environment.

This is acceptable for this learning lab.

For production Jenkins, use stronger isolation and dedicated Kubernetes credentials with only the permissions Jenkins actually needs.

Do not use:

```text
cluster-admin
```

permissions unnecessarily.

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
Kubernetes Namespace
Deployment
ReplicaSet
Pods
Service
Rolling Update
Rollout
Rollback
Jenkins Kubernetes Deployment
Jenkins + Docker + Kubernetes
```

---

# Next Lab

## Lab 11 — Complete Jenkins CI/CD Pipeline

Now we will combine everything into one proper pipeline:

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
    v
Application
```

This will be our first complete **GitHub → Jenkins → Docker Hub → Kubernetes CI/CD pipeline**.

# End of Lab 10
