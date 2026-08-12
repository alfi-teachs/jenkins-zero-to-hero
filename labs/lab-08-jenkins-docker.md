# Jenkins Zero to Hero — Lab 8

## Jenkins + Docker

This lab continues from:

```text
Lab 1 → Jenkins Installation
Lab 2 → Jenkins Jobs, Workspace and Parameters
Lab 3 → Jenkins + Git + GitHub
Lab 4 → GitHub Webhooks
Lab 5 → Jenkins Pipeline and Jenkinsfile
Lab 6 → Jenkinsfile Deep Dive
Lab 7 → Jenkins CI Pipeline
```

In this lab, Jenkins will build a Docker image from the application source code.

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
    +---- Validate
    |
    +---- Build
    |
    +---- Test
    |
    +---- Docker Build
    |
    +---- Docker Test
    |
    v
Docker Image
```

Jenkins' official Docker documentation notes that the standard `jenkins/jenkins` image does not include the Docker CLI, so we will create a small custom Jenkins image with Docker CLI support. Jenkins' Pipeline Docker documentation also describes communicating with a Docker daemon from a pipeline. :contentReference[oaicite:0]{index=0}

---

# Lab Objective

By the end of this lab, you will understand:

```text
Docker from Jenkins
Docker CLI inside Jenkins
Jenkins custom Docker image
Docker socket
Dockerfile
Docker Build
Docker Image
Docker Container
Docker Test
Docker Image Tags
Jenkins + Docker Pipeline
```

---

# Important

In the previous labs, your Jenkins container was started from:

```text
jenkins/jenkins:lts
```

That standard image does not include the Docker CLI.

For this lab, we will create:

```text
jenkins-docker:lts
```

This image will contain:

```text
Jenkins
+
Docker CLI
```

We will preserve your existing Jenkins data using:

```text
jenkins_home
```

Your existing Jenkins jobs and configuration will therefore remain available.

---

# Prerequisites

Make sure Docker Desktop is running.

Check:

```bash
docker ps
```

Check Docker:

```bash
docker version
```

Check Jenkins:

```bash
docker ps --filter "name=jenkins"
```

You should see:

```text
jenkins
```

---

# Part 1 — Go to the Jenkins Repository

Open Git Bash.

Run:

```bash
cd /c/project/jenkins-zero-to-hero
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
README.md
labs
```

---

# Part 2 — Create the Lab 8 Markdown File

Go into the labs folder:

```bash
cd labs
```

Create:

```bash
touch lab-08-jenkins-docker.md
```

Check:

```bash
ls
```

You should see:

```text
lab-01-install-jenkins.md
lab-02-jenkins-jobs-workspace.md
lab-03-jenkins-git-github.md
lab-04-jenkins-webhooks.md
lab-05-jenkins-pipeline.md
lab-06-jenkinsfile-deep-dive.md
lab-07-jenkins-ci-pipeline.md
lab-08-jenkins-docker.md
```

---

# Part 3 — Verify the Jenkins Volume

Before changing the Jenkins container, verify that your Jenkins data is stored in the Docker volume.

Run:

```bash
docker volume ls
```

You should see:

```text
jenkins_home
```

Inspect:

```bash
docker volume inspect jenkins_home
```

The exact output will be different.

---

# Part 4 — Verify the Jenkins Network

Run:

```bash
docker network ls
```

You should see:

```text
jenkins
```

If the network does not exist:

```bash
docker network create jenkins
```

Check again:

```bash
docker network ls
```

Expected:

```text
NETWORK ID     NAME       DRIVER    SCOPE
xxxxxxxxxxxx   jenkins    bridge    local
```

---

# Part 5 — Verify the Current Jenkins Container

Run:

```bash
docker ps --filter "name=jenkins"
```

Check the Jenkins image:

```bash
docker inspect jenkins --format "{{.Config.Image}}"
```

You may see:

```text
jenkins/jenkins:lts
```

---

# Part 6 — Check Whether Docker CLI Exists in Jenkins

Run:

```bash
docker exec jenkins docker --version
```

You may get an error similar to:

```text
exec: "docker": executable file not found
```

This is expected with the standard Jenkins image because Docker CLI is not included. :contentReference[oaicite:1]{index=1}

---

# Part 7 — Create a Custom Jenkins Directory

Go to your project directory:

```bash
cd /c/project
```

Create:

```bash
mkdir jenkins-docker
```

Enter:

```bash
cd jenkins-docker
```

Check:

```bash
pwd
```

---

# Part 8 — Create the Jenkins Dockerfile

Create:

```bash
touch Dockerfile
```

Open:

```bash
code Dockerfile
```

Paste:

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

USER jenkins
```

Save the file.

This follows the basic approach in Jenkins' official Docker documentation: customize the Jenkins image and install the Docker CLI. :contentReference[oaicite:2]{index=2}

---

# Part 9 — Check the Dockerfile

Run:

```bash
cat Dockerfile
```

You should see:

```text
FROM jenkins/jenkins:lts
USER root
...
USER jenkins
```

---

# Part 10 — Build the Custom Jenkins Image

Run:

```bash
docker build -t jenkins-docker:lts .
```

Docker will build the image.

When finished:

```text
Successfully tagged jenkins-docker:lts
```

Check the image:

```bash
docker images | grep jenkins-docker
```

Expected:

```text
jenkins-docker   lts
```

---

# Part 11 — Stop the Existing Jenkins Container

Do not remove the volume.

First stop Jenkins:

```bash
docker stop jenkins
```

Check:

```bash
docker ps
```

The Jenkins container should no longer be running.

---

# Part 12 — Remove Only the Old Jenkins Container

Remove the container:

```bash
docker rm jenkins
```

Check:

```bash
docker ps -a
```

You should not see the old `jenkins` container.

---

# Part 13 — IMPORTANT: Do Not Delete the Jenkins Volume

Do not run:

```bash
docker volume rm jenkins_home
```

Your volume contains:

```text
Jenkins Jobs
Jenkins Configuration
Plugins
Build History
Credentials
Users
```

We are keeping it.

---

# Part 14 — Start Jenkins with Docker Support

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
  jenkins-docker:lts
```

The important Docker socket mapping is:

```text
/var/run/docker.sock
```

Jenkins' Docker Pipeline documentation notes that the Docker Pipeline functionality commonly communicates with a local Docker daemon through this socket. :contentReference[oaicite:3]{index=3}

---

# Part 15 — Check Jenkins

Run:

```bash
docker ps
```

You should see:

```text
jenkins
```

Check the image:

```bash
docker inspect jenkins --format "{{.Config.Image}}"
```

Expected:

```text
jenkins-docker:lts
```

---

# Part 16 — Check Jenkins Logs

Run:

```bash
docker logs jenkins --tail 50
```

You want to see Jenkins starting normally.

---

# Part 17 — Open Jenkins

Open:

```text
http://localhost:8081
```

Your existing Jenkins configuration should still be there.

You should see your previous jobs such as:

```text
hello-jenkins
lab-02-job
lab-03-git-checkout
lab-04-github-webhook
lab-05-pipeline
```

The exact jobs depend on what you created.

---

# Part 18 — Verify Docker CLI Inside Jenkins

Now run:

```bash
docker exec jenkins docker --version
```

Expected:

```text
Docker version ...
```

The exact version may be different.

This confirms:

```text
Jenkins
   |
   v
Docker CLI
```

---

# Part 19 — Test Docker Access from Jenkins

Run:

```bash
docker exec jenkins docker ps
```

You should see the containers running on the Docker engine.

For example:

```text
CONTAINER ID
IMAGE
COMMAND
STATUS
NAMES
```

This proves Jenkins can communicate with Docker.

---

# Part 20 — Check Docker Information

Run:

```bash
docker exec jenkins docker info
```

You should receive Docker engine information.

If this command works, Jenkins has access to the Docker daemon.

---

# Part 21 — Understand What Happened

Your setup is now:

```text
Windows / Docker Desktop
        |
        v
Docker Engine
        |
        +----------------+
        |                |
        v                v
     Jenkins         Other Containers
        |
        |
        v
Docker CLI
```

The Jenkins container can use the Docker CLI to communicate with the Docker daemon.

---

# Part 22 — Go to the Application Repository

Open Git Bash:

```bash
cd /c/project/jenkins-demo-app
```

Check:

```bash
ls
```

You should have:

```text
Jenkinsfile
README.md
app.sh
version.txt
```

---

# Part 23 — Create a Dockerfile for the Application

Create:

```bash
touch Dockerfile
```

Check:

```bash
ls
```

Now you should have:

```text
Dockerfile
Jenkinsfile
README.md
app.sh
version.txt
```

---

# Part 24 — Open the Application Dockerfile

Run:

```bash
code Dockerfile
```

Paste:

```dockerfile
FROM alpine:3.20

WORKDIR /app

COPY app.sh .
COPY version.txt .

RUN chmod +x app.sh

CMD ["./app.sh"]
```

Save.

---

# Part 25 — Understand the Dockerfile

```text
FROM alpine:3.20
```

Uses Alpine Linux as the base image.

```text
WORKDIR /app
```

Creates the working directory:

```text
/app
```

```text
COPY app.sh .
COPY version.txt .
```

Copies application files into the image.

```text
RUN chmod +x app.sh
```

Makes the script executable.

```text
CMD ["./app.sh"]
```

Runs the application when the container starts.

---

# Part 26 — Test the Docker Build Locally

Before using Jenkins, build the image manually.

Run:

```bash
docker build -t jenkins-demo-app:local .
```

Check:

```bash
docker images | grep jenkins-demo-app
```

Expected:

```text
jenkins-demo-app   local
```

---

# Part 27 — Run the Docker Container Locally

Run:

```bash
docker run --rm jenkins-demo-app:local
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

# Part 28 — Remove the Local Docker Image

We will later let Jenkins build the image.

For now, remove the local test image:

```bash
docker rmi jenkins-demo-app:local
```

Check:

```bash
docker images | grep jenkins-demo-app
```

There should be no `local` image.

---

# Part 29 — Update the Jenkinsfile

Open:

```bash
code Jenkinsfile
```

Replace the Jenkinsfile with:

```groovy
pipeline {

    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    parameters {
        string(
            name: 'APP_VERSION',
            defaultValue: '1.0',
            description: 'Application version'
        )

        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'test', 'prod'],
            description: 'Target environment'
        )
    }

    environment {
        APP_NAME = 'jenkins-demo-app'
        IMAGE_NAME = 'jenkins-demo-app'
        BUILD_OUTPUT = 'build'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '======================================'
                echo 'CHECKOUT'
                echo '======================================'

                checkout scm

                sh 'git log --oneline -1'
            }
        }

        stage('Validate') {
            steps {
                echo '======================================'
                echo 'VALIDATE'
                echo '======================================'

                sh 'test -f Jenkinsfile'
                sh 'test -f Dockerfile'
                sh 'test -f app.sh'
                sh 'test -f version.txt'
                sh 'test -f README.md'

                echo 'Required files are present.'
            }
        }

        stage('Application Test') {
            steps {
                echo '======================================'
                echo 'APPLICATION TEST'
                echo '======================================'

                sh 'chmod +x app.sh'

                sh 'grep -q "JENKINS DEMO APPLICATION" app.sh'
                sh 'grep -q "Application is running successfully." app.sh'

                echo 'Application tests passed.'
            }
        }

        stage('Docker Build') {
            steps {
                echo '======================================'
                echo 'DOCKER BUILD'
                echo '======================================'

                sh 'docker version'

                sh 'docker build -t "$IMAGE_NAME:$APP_VERSION" .'

                sh 'docker image ls "$IMAGE_NAME"'
            }
        }

        stage('Docker Test') {
            steps {
                echo '======================================'
                echo 'DOCKER TEST'
                echo '======================================'

                sh 'docker run --rm "$IMAGE_NAME:$APP_VERSION"'

                echo 'Docker container test completed.'
            }
        }

        stage('Package') {
            steps {
                echo '======================================'
                echo 'PACKAGE'
                echo '======================================'

                sh 'rm -rf "$BUILD_OUTPUT"'
                sh 'mkdir -p "$BUILD_OUTPUT"'

                sh 'echo "Application=$APP_NAME" > "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "Version=$APP_VERSION" >> "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "Environment=$ENVIRONMENT" >> "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "DockerImage=$IMAGE_NAME:$APP_VERSION" >> "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "JenkinsBuild=$BUILD_NUMBER" >> "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "GitCommit=$(git rev-parse HEAD)" >> "$BUILD_OUTPUT"/build-info.txt'

                sh 'cat "$BUILD_OUTPUT"/build-info.txt'
            }
        }

        stage('Archive') {
            steps {
                echo '======================================'
                echo 'ARCHIVE'
                echo '======================================'

                archiveArtifacts artifacts: 'build/**', fingerprint: true

                echo 'Build information archived.'
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'DOCKER CI PIPELINE SUCCESS'
            echo '======================================'

            echo "Docker image: ${IMAGE_NAME}:${params.APP_VERSION}"
            echo "Environment: ${params.ENVIRONMENT}"
        }

        failure {
            echo '======================================'
            echo 'DOCKER CI PIPELINE FAILED'
            echo '======================================'
        }

        always {
            echo '======================================'
            echo 'PIPELINE FINISHED'
            echo '======================================'
        }
    }
}
```

Save.

---

# Part 30 — Understand the Docker Build Stage

This command:

```bash
docker build -t "$IMAGE_NAME:$APP_VERSION" .
```

creates an image.

For example:

```text
jenkins-demo-app:1.0
```

The structure is:

```text
Image Name:
jenkins-demo-app

Tag:
1.0

Full Image:
jenkins-demo-app:1.0
```

---

# Part 31 — Understand the Docker Test Stage

This command:

```bash
docker run --rm "$IMAGE_NAME:$APP_VERSION"
```

starts a temporary container.

The container runs:

```text
app.sh
```

and then exits.

Because we use:

```text
--rm
```

Docker removes the container after it exits.

---

# Part 32 — Check the Jenkinsfile

Run:

```bash
cat Jenkinsfile
```

Check that the pipeline contains:

```text
Checkout
Validate
Application Test
Docker Build
Docker Test
Package
Archive
```

---

# Part 33 — Check the Application Dockerfile

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

# Part 34 — Test Dockerfile Locally Again

Run:

```bash
docker build -t jenkins-demo-app:test .
```

Check:

```bash
docker images | grep jenkins-demo-app
```

Run:

```bash
docker run --rm jenkins-demo-app:test
```

---

# Part 35 — Remove Local Test Image

Run:

```bash
docker rmi jenkins-demo-app:test
```

Check:

```bash
docker images | grep jenkins-demo-app
```

---

# Part 36 — Commit Dockerfile and Jenkinsfile

Check:

```bash
git status
```

You should see:

```text
modified: Jenkinsfile
untracked: Dockerfile
```

Add:

```bash
git add Jenkinsfile Dockerfile
```

Check:

```bash
git status
```

Commit:

```bash
git commit -m "Add Jenkins Docker build pipeline"
```

---

# Part 37 — Push to GitHub

Run:

```bash
git push origin main
```

---

# Part 38 — Open Jenkins

Open:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

Because the job uses:

```text
Pipeline script from SCM
```

Jenkins will retrieve the new Jenkinsfile.

---

# Part 39 — Run the Pipeline

Click:

```text
Build with Parameters
```

Enter:

```text
APP_VERSION:
1.0
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

# Part 40 — Watch the Pipeline

The pipeline should now execute:

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
Package
    |
    v
Archive
```

---

# Part 41 — Check Console Output

Open the new build:

```text
Console Output
```

You should see:

```text
DOCKER BUILD
```

Then Docker information.

Then:

```text
Successfully built
```

or equivalent Docker build output.

Then:

```text
DOCKER TEST
```

Then the application output:

```text
JENKINS DEMO APPLICATION
```

Finally:

```text
DOCKER CI PIPELINE SUCCESS
```

---

# Part 42 — Check Docker Images Created by Jenkins

From Git Bash:

```bash
docker images | grep jenkins-demo-app
```

You should see something similar to:

```text
jenkins-demo-app   1.0
```

The exact image ID and creation time will be different.

---

# Part 43 — Inspect the Docker Image

Run:

```bash
docker image inspect jenkins-demo-app:1.0
```

You can inspect:

```text
Image ID
Architecture
OS
Created time
Entrypoint
Command
Layers
```

---

# Part 44 — Run the Jenkins-Built Image

Run:

```bash
docker run --rm jenkins-demo-app:1.0
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

# Part 45 — List Docker Containers

Run:

```bash
docker ps
```

The Jenkins container should still be running.

The application test container should normally not appear because the pipeline uses:

```text
--rm
```

Check stopped containers:

```bash
docker ps -a
```

You should not have leftover application test containers from the pipeline.

---

# Part 46 — Check Jenkins Workspace

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline"
```

You should see:

```text
Dockerfile
Jenkinsfile
README.md
app.sh
version.txt
build
```

---

# Part 47 — Check Build Information

Run:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-05-pipeline/build/build-info.txt"
```

Expected format:

```text
Application=jenkins-demo-app
Version=1.0
Environment=dev
DockerImage=jenkins-demo-app:1.0
JenkinsBuild=<build-number>
GitCommit=<commit-hash>
```

---

# Part 48 — Run Another Docker Build

Now test another version.

Open Jenkins:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
2.0
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

# Part 49 — Check Docker Images

Run:

```bash
docker images | grep jenkins-demo-app
```

You should now have both:

```text
jenkins-demo-app   1.0
jenkins-demo-app   2.0
```

---

# Part 50 — Run Version 2.0

Run:

```bash
docker run --rm jenkins-demo-app:2.0
```

The application should run successfully.

---

# Part 51 — Test Production Tag

Run the Jenkins pipeline again:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
3.0
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

After the build completes:

```bash
docker images | grep jenkins-demo-app
```

You should now see:

```text
jenkins-demo-app   1.0
jenkins-demo-app   2.0
jenkins-demo-app   3.0
```

---

# Part 52 — Understand Image Tags

We now have:

```text
jenkins-demo-app:1.0
jenkins-demo-app:2.0
jenkins-demo-app:3.0
```

The tag identifies the version.

For example:

```text
jenkins-demo-app:3.0
```

means:

```text
Image:
jenkins-demo-app

Tag:
3.0
```

Later, we will push these images to Docker Hub.

---

# Part 53 — Check Docker Images from Jenkins

Run:

```bash
docker exec jenkins docker images
```

You should see the Docker images Jenkins created.

---

# Part 54 — Check Docker Containers from Jenkins

Run:

```bash
docker exec jenkins docker ps
```

Jenkins should be able to see the Docker engine.

---

# Part 55 — Check Docker Build Cache

Run:

```bash
docker exec jenkins docker system df
```

This shows Docker disk usage.

You may see:

```text
Images
Containers
Local Volumes
Build Cache
```

---

# Part 56 — Clean Unused Docker Images

Do not delete images you want to keep.

To see dangling images:

```bash
docker images --filter "dangling=true"
```

Remove dangling images only:

```bash
docker image prune
```

Docker may ask for confirmation.

Type:

```text
y
```

---

# Part 57 — Remove Specific Test Images

For example:

```bash
docker rmi jenkins-demo-app:1.0
```

Do not run this if you want to keep version `1.0`.

You can remove version `2.0`:

```bash
docker rmi jenkins-demo-app:2.0
```

Again, only do this if you do not need that image.

---

# Part 58 — Understand Jenkins + Docker Architecture

Your current architecture is:

```text
                    Docker Desktop
                         |
                    Docker Engine
                         |
          +--------------+--------------+
          |                             |
          v                             v
       Jenkins                    Other Containers
          |
          | Docker CLI
          |
          v
   Docker Socket
          |
          v
     Docker Engine
          |
          v
 Docker Build / Run
          |
          v
 jenkins-demo-app:1.0
```

---

# Part 59 — Understand the Docker Socket

The Jenkins container has access to:

```text
/var/run/docker.sock
```

This allows the Docker CLI inside Jenkins to communicate with the Docker daemon.

This is powerful access.

In production environments, Docker socket access should be treated as a high-privilege capability.

For this local learning lab, we use it to understand how Jenkins can control Docker. Jenkins' documentation also discusses Docker daemon access and alternative Docker server configurations. :contentReference[oaicite:4]{index=4}

---

# Part 60 — Test a Docker Command Directly Inside Jenkins

Run:

```bash
docker exec jenkins docker run --rm hello-world
```

Expected output includes:

```text
Hello from Docker!
```

This is an important test.

It proves:

```text
Jenkins Container
        |
        v
Docker CLI
        |
        v
Docker Engine
        |
        v
Docker Container
```

---

# Part 61 — Check Jenkins Docker CLI

Run:

```bash
docker exec jenkins which docker
```

Expected:

```text
/usr/bin/docker
```

The exact path may differ.

Check version:

```bash
docker exec jenkins docker version
```

---

# Part 62 — Test Dockerfile Changes

Change the application version.

Open:

```bash
code version.txt
```

Change the version to:

```text
8.0
```

Save.

---

# Part 63 — Commit Version 8.0

Run:

```bash
git add version.txt
```

Commit:

```bash
git commit -m "Prepare Docker build version 8.0"
```

Push:

```bash
git push origin main
```

---

# Part 64 — Trigger Jenkins

If your GitHub webhook from Lab 4 is active:

```text
GitHub
    |
    v
Webhook
    |
    v
Jenkins
```

The pipeline should start automatically.

If the webhook is not active, open:

```text
http://localhost:8081
```

and click:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
8.0
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

# Part 65 — Verify Docker Image 8.0

Run:

```bash
docker images | grep jenkins-demo-app
```

You should see:

```text
jenkins-demo-app   8.0
```

Run:

```bash
docker run --rm jenkins-demo-app:8.0
```

The application should run.

---

# Part 66 — Check the Final Jenkins Build

Open the latest Jenkins build.

Verify:

```text
Checkout
Validate
Application Test
Docker Build
Docker Test
Package
Archive
```

You should see:

```text
DOCKER CI PIPELINE SUCCESS
```

---

# Part 67 — Understand the CI + Docker Pipeline

The pipeline is now:

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
    +---- Application Test
    |
    +---- Docker Build
    |
    +---- Docker Test
    |
    +---- Package
    |
    +---- Archive
    |
    v
Docker Image
```

This is the foundation for:

```text
Jenkins
    |
    v
Docker Image
    |
    v
Docker Registry
    |
    v
Kubernetes
```

---

# Part 68 — Check Git Status

Run:

```bash
cd /c/project/jenkins-demo-app
```

Then:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# Part 69 — Check Git History

Run:

```bash
git log --oneline --max-count=10
```

You should see commits related to:

```text
Prepare Docker build version 8.0
Add Jenkins Docker build pipeline
```

Your exact commit hashes and history may be different.

---

# Part 70 — Save Lab 8 Documentation

Go to the Jenkins documentation repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go to labs:

```bash
cd labs
```

Open:

```bash
code lab-08-jenkins-docker.md
```

Paste this complete lab into the file.

Save.

---

# Part 71 — Check Git Status

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
new file: labs/lab-08-jenkins-docker.md
```

or:

```text
modified: labs/lab-08-jenkins-docker.md
```

---

# Part 72 — Add Lab 8

```bash
git add labs/lab-08-jenkins-docker.md
```

Check:

```bash
git status
```

---

# Part 73 — Commit Lab 8

```bash
git commit -m "Add Jenkins Lab 8 Docker integration"
```

---

# Part 74 — Push Lab 8

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

# Part 75 — Verify Jenkins Repository

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
    └── lab-08-jenkins-docker.md
```

---

# Lab 8 Completion Checklist

```text
[ ] Docker Desktop is running
[ ] Jenkins Docker volume verified
[ ] Jenkins Docker network verified
[ ] Existing Jenkins container verified
[ ] Standard Jenkins Docker CLI checked
[ ] Custom Jenkins Dockerfile created
[ ] Custom Jenkins image built
[ ] Old Jenkins container stopped
[ ] Old Jenkins container removed
[ ] Jenkins volume preserved
[ ] Custom Jenkins container started
[ ] Jenkins restored successfully
[ ] Docker CLI works inside Jenkins
[ ] Docker Engine accessible from Jenkins
[ ] Application Dockerfile created
[ ] Docker image built locally
[ ] Docker container tested locally
[ ] Jenkinsfile updated for Docker
[ ] Docker Build stage created
[ ] Docker Test stage created
[ ] Jenkins successfully built Docker image
[ ] Jenkins successfully ran Docker container
[ ] Docker image verified
[ ] Docker image tags tested
[ ] Multiple application versions built
[ ] Docker image inspected
[ ] Docker CLI tested from Jenkins
[ ] hello-world tested from Jenkins
[ ] CI + Docker workflow verified
[ ] Lab 8 documentation saved
[ ] Lab 8 committed
[ ] Lab 8 pushed to GitHub
```

---

# Lab 8 Cleanup

## Recommended

Do not remove Jenkins if you are continuing to Lab 9.

Keep:

```text
jenkins container
jenkins_home volume
jenkins network
```

Keep your application repository:

```text
jenkins-demo-app
```

---

# Remove Specific Application Images

See images:

```bash
docker images | grep jenkins-demo-app
```

Remove a specific version:

```bash
docker rmi jenkins-demo-app:1.0
```

Remove another version:

```bash
docker rmi jenkins-demo-app:2.0
```

Remove another version:

```bash
docker rmi jenkins-demo-app:3.0
```

Remove version 8.0 if desired:

```bash
docker rmi jenkins-demo-app:8.0
```

Only remove images you no longer need.

---

# Remove Dangling Images

Check:

```bash
docker images --filter "dangling=true"
```

If there are dangling images:

```bash
docker image prune
```

---

# Important

Do not remove:

```text
jenkins_home
```

because it contains Jenkins data.

Do not run:

```bash
docker volume rm jenkins_home
```

unless you intentionally want to delete all Jenkins data.

---

# Complete Jenkins Cleanup

Only use this when you want to completely remove Jenkins.

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
Jenkins CI + Docker
```

---

# Next Lab

## Lab 9 — Jenkins + Docker Hub

The next step is to take the Docker image Jenkins built and push it to a container registry.

The workflow will become:

```text
Developer
    |
    | git push
    v
GitHub
    |
    v
Jenkins
    |
    +---- Checkout
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
    v
Docker Hub
```

We will learn:

```text
Docker Hub
Registry
Image Tags
Jenkins Credentials
Docker Login
Docker Push
Credential Security
```

# End of Lab 8
