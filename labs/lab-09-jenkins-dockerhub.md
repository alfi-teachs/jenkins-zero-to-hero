# Jenkins Zero to Hero — Lab 9

## Jenkins + Docker Hub

This lab continues from:

```text
Lab 1 → Jenkins Installation
Lab 2 → Jenkins Jobs, Workspace and Parameters
Lab 3 → Jenkins + Git + GitHub
Lab 4 → GitHub Webhooks
Lab 5 → Jenkins Pipeline and Jenkinsfile
Lab 6 → Jenkinsfile Deep Dive
Lab 7 → Jenkins CI Pipeline
Lab 8 → Jenkins + Docker
```

In Lab 8, Jenkins built a Docker image locally.

In this lab, Jenkins will:

```text
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

Docker recommends using Personal Access Tokens (PATs) instead of passwords for automated Docker CLI and CI/CD authentication. PATs can be given appropriate permissions and should be stored securely rather than committed to source code. :contentReference[oaicite:0]{index=0}

Jenkins stores credentials securely and pipelines should reference the Jenkins credential by its ID rather than putting the secret directly in the Jenkinsfile. :contentReference[oaicite:1]{index=1}

---

# Lab Objective

By the end of this lab, you will understand:

```text
Docker Hub
Docker Registry
Docker Repository
Docker Image Tags
Docker Hub Personal Access Token
Jenkins Credentials
Docker Login
Docker Push
Docker Pull
Jenkins + Docker Hub
Secure Credential Usage
```

---

# Important

We will use:

```text
Docker Hub
```

as our container registry.

We will create a Docker Hub repository:

```text
jenkins-demo-app
```

Your full image name will be:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

For example:

```text
alfia/jenkins-demo-app
```

Use your actual Docker Hub username.

Do not use your GitHub username unless they are the same.

---

# Prerequisites

Jenkins must be running.

Check:

```bash
docker ps
```

Expected:

```text
jenkins
```

Check Docker:

```bash
docker version
```

Check Jenkins Docker CLI:

```bash
docker exec jenkins docker --version
```

You should get a Docker version.

---

# Part 1 — Check the Application Repository

Go to the application repository:

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

---

# Part 2 — Check Git Status

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

# Part 3 — Check GitHub Remote

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

# Part 4 — Create Docker Hub Repository

Open:

```text
https://hub.docker.com/
```

Sign in to your Docker Hub account.

Create a new repository.

Use:

```text
Repository name:
jenkins-demo-app
```

For this lab, you can use:

```text
Visibility:
Public
```

The repository should become:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

For example:

```text
alfia/jenkins-demo-app
```

---

# Part 5 — Verify the Docker Hub Repository

Open:

```text
https://hub.docker.com/u/YOUR-DOCKERHUB-USERNAME
```

You should see:

```text
jenkins-demo-app
```

---

# Part 6 — Create a Docker Hub Personal Access Token

Do not use your normal Docker Hub password in Jenkins.

Docker recommends Personal Access Tokens for CI/CD authentication. :contentReference[oaicite:2]{index=2}

Open Docker Hub.

Go to:

```text
Account Settings
→ Personal access tokens
```

Create a new token.

Use a name such as:

```text
jenkins-ci
```

For permissions, you need permission to push images.

Use:

```text
Read & Write
```

If Docker Hub presents more specific repository-scoped options, choose permissions that allow the Jenkins pipeline to push to:

```text
jenkins-demo-app
```

Generate the token.

---

# Part 7 — Save the Docker Hub Token

Docker only shows the token when it is generated.

Copy it immediately.

Store it somewhere secure temporarily.

Do not put it in:

```text
Jenkinsfile
GitHub
README.md
Dockerfile
```

Docker specifically warns that access tokens should be treated like passwords and not committed to source control. :contentReference[oaicite:3]{index=3}

---

# Part 8 — Test Docker Hub Login Locally

Do not put your token directly into the shell command if you can avoid it.

Use:

```bash
docker login --username YOUR-DOCKERHUB-USERNAME
```

Docker will prompt for:

```text
Password:
```

Paste your Docker Hub Personal Access Token.

Expected:

```text
Login Succeeded
```

Docker documents username + PAT authentication for CLI access. :contentReference[oaicite:4]{index=4}

---

# Part 9 — Check Docker Login

Run:

```bash
docker info
```

If your Docker CLI is authenticated, the Docker client should be able to use the stored registry credentials.

Do not publish or commit your Docker configuration containing credentials.

---

# Part 10 — Create a Local Docker Image

Go to the application repository:

```bash
cd /c/project/jenkins-demo-app
```

Check the current application version:

```bash
cat version.txt
```

For this lab, use:

```text
9.0
```

If your version is different, that is okay.

Build:

```bash
docker build -t YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:9.0 .
```

Check:

```bash
docker images | grep jenkins-demo-app
```

You should see:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app   9.0
```

---

# Part 11 — Test the Image

Run:

```bash
docker run --rm YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:9.0
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

# Part 12 — Push the Image Manually

Before Jenkins does it automatically, understand the Docker commands manually.

Run:

```bash
docker push YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:9.0
```

Docker should upload the image layers.

At the end, you should see a digest similar to:

```text
digest: sha256:xxxxxxxxxxxxxxxx
```

The exact digest will be different.

---

# Part 13 — Verify the Image in Docker Hub

Open:

```text
https://hub.docker.com/
```

Open:

```text
jenkins-demo-app
```

You should see the tag:

```text
9.0
```

Now you know the manual process:

```text
Docker Build
    |
    v
Docker Image
    |
    v
Docker Login
    |
    v
Docker Push
    |
    v
Docker Hub
```

---

# Part 14 — Pull the Image from Docker Hub

To test that the registry contains your image, remove the local image first.

Run:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:9.0
```

Check:

```bash
docker images | grep jenkins-demo-app
```

The `9.0` image should no longer be present.

Now pull it:

```bash
docker pull YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:9.0
```

Check:

```bash
docker images | grep jenkins-demo-app
```

You should see:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app   9.0
```

Run it:

```bash
docker run --rm YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:9.0
```

This proves:

```text
Docker Hub
    |
    v
Docker Pull
    |
    v
Local Docker
```

---

# Part 15 — Jenkins Credentials

Now we will store the Docker Hub credentials in Jenkins.

Open:

```text
http://localhost:8081
```

Go to:

```text
Manage Jenkins
```

Then:

```text
Credentials
```

Open:

```text
Global
```

Select:

```text
Global credentials
```

Click:

```text
Add Credentials
```

---

# Part 16 — Create the Docker Hub Credential

For:

```text
Kind
```

select:

```text
Username with password
```

For:

```text
Username
```

enter:

```text
YOUR-DOCKERHUB-USERNAME
```

For:

```text
Password
```

paste your:

```text
Docker Hub Personal Access Token
```

For:

```text
ID
```

enter:

```text
dockerhub-credentials
```

For:

```text
Description
```

enter:

```text
Docker Hub credentials for Jenkins
```

Click:

```text
Create
```

Jenkins will store the credentials and the pipeline will reference them using the credential ID. :contentReference[oaicite:5]{index=5}

---

# Part 17 — Verify Jenkins Credential

Go to:

```text
Manage Jenkins
→ Credentials
→ Global
```

You should see something similar to:

```text
dockerhub-credentials
```

Do not click to expose or copy the token.

---

# Part 18 — Important Security Rule

Never write this in your Jenkinsfile:

```groovy
docker login -u myusername -p mypassword
```

Never write this:

```groovy
DOCKER_PASSWORD = 'mypassword'
```

Never commit a token:

```text
dckr_pat_xxxxxxxxx
```

Instead:

```text
Jenkins Credentials
        |
        v
Credential ID
        |
        v
Pipeline
        |
        v
Docker Login
```

Jenkins credentials are stored securely and pipelines use the credential ID. :contentReference[oaicite:6]{index=6}

---

# Part 19 — Prepare the Jenkinsfile

Go to:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

Replace it with:

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
            description: 'Docker image version'
        )

        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'test', 'prod'],
            description: 'Target environment'
        )
    }

    environment {
        DOCKERHUB_USERNAME = 'YOUR-DOCKERHUB-USERNAME'
        IMAGE_NAME = 'jenkins-demo-app'
        IMAGE = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}"
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

                sh 'test -f Dockerfile'
                sh 'test -f Jenkinsfile'
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

                sh 'docker build -t "$IMAGE:$APP_VERSION" .'

                sh 'docker image inspect "$IMAGE:$APP_VERSION" > /dev/null'

                echo "Docker image created: ${IMAGE}:${params.APP_VERSION}"
            }
        }

        stage('Docker Test') {
            steps {

                echo '======================================'
                echo 'DOCKER TEST'
                echo '======================================'

                sh 'docker run --rm "$IMAGE:$APP_VERSION"'

                echo 'Docker image test completed.'
            }
        }

        stage('Docker Login') {
            steps {

                echo '======================================'
                echo 'DOCKER LOGIN'
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

                echo 'Docker Hub authentication completed.'
            }
        }

        stage('Docker Push') {
            steps {

                echo '======================================'
                echo 'DOCKER PUSH'
                echo '======================================'

                sh 'docker push "$IMAGE:$APP_VERSION"'

                echo "Image pushed: ${IMAGE}:${params.APP_VERSION}"
            }
        }

        stage('Create Build Information') {
            steps {

                echo '======================================'
                echo 'BUILD INFORMATION'
                echo '======================================'

                sh 'rm -rf build'
                sh 'mkdir -p build'

                sh 'echo "Application=$IMAGE_NAME" > build/build-info.txt'
                sh 'echo "DockerImage=$IMAGE:$APP_VERSION" >> build/build-info.txt'
                sh 'echo "Environment=$ENVIRONMENT" >> build/build-info.txt'
                sh 'echo "JenkinsBuild=$BUILD_NUMBER" >> build/build-info.txt'
                sh 'echo "GitCommit=$(git rev-parse HEAD)" >> build/build-info.txt'
                sh 'echo "GitBranch=$(git branch --show-current)" >> build/build-info.txt'

                sh 'cat build/build-info.txt'
            }
        }

        stage('Archive') {
            steps {

                echo '======================================'
                echo 'ARCHIVE'
                echo '======================================'

                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
        }
    }

    post {

        success {

            echo '======================================'
            echo 'DOCKER HUB PIPELINE SUCCESS'
            echo '======================================'

            echo "Image: ${IMAGE}:${params.APP_VERSION}"
            echo "Environment: ${params.ENVIRONMENT}"

        }

        failure {

            echo '======================================'
            echo 'PIPELINE FAILED'
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

Replace:

```text
YOUR-DOCKERHUB-USERNAME
```

with your actual Docker Hub username.

For example:

```groovy
DOCKERHUB_USERNAME = 'alfia'
```

Do not put the PAT in this file.

---

# Part 20 — Understand the Credential Block

This part:

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'dockerhub-credentials',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_TOKEN'
    )
])
```

temporarily makes the Jenkins credentials available to the build.

The pipeline then uses:

```bash
$DOCKER_USER
```

and:

```bash
$DOCKER_TOKEN
```

Jenkins handles the credential rather than storing the secret in the Jenkinsfile. :contentReference[oaicite:7]{index=7}

---

# Part 21 — Understand Docker Login

The pipeline uses:

```bash
echo "$DOCKER_TOKEN" | docker login \
    --username "$DOCKER_USER" \
    --password-stdin
```

The token is passed through standard input instead of putting it directly in the command line.

Docker documents `--password-stdin` for supplying a password or PAT to `docker login`. :contentReference[oaicite:8]{index=8}

---

# Part 22 — Check the Jenkinsfile

Run:

```bash
cat Jenkinsfile
```

Check that you have:

```text
Checkout
Validate
Application Test
Docker Build
Docker Test
Docker Login
Docker Push
Create Build Information
Archive
```

---

# Part 23 — Check the Dockerfile

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

# Part 24 — Check Git Status

Run:

```bash
git status
```

You should see:

```text
modified: Jenkinsfile
```

---

# Part 25 — Commit the Jenkinsfile

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add Docker Hub push to Jenkins pipeline"
```

---

# Part 26 — Push the Jenkinsfile

Run:

```bash
git push origin main
```

---

# Part 27 — Open Jenkins

Open:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

The Jenkins job should use:

```text
Pipeline script from SCM
```

and:

```text
Jenkinsfile
```

from:

```text
*/main
```

---

# Part 28 — Run the Pipeline

Click:

```text
Build with Parameters
```

Enter:

```text
APP_VERSION:
10.0
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

# Part 29 — Watch the Pipeline

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
Build Information
    |
    v
Archive
```

---

# Part 30 — Check Docker Login

Open:

```text
Console Output
```

You should see something indicating successful authentication.

For example:

```text
Login Succeeded
```

The Docker token itself should not appear in the console.

---

# Part 31 — Check Docker Push

Console output should show Docker pushing layers.

You should see output similar to:

```text
The push refers to repository
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

You should also see a digest.

The exact digest will be different.

---

# Part 32 — Verify Pipeline Success

At the end you should see:

```text
DOCKER HUB PIPELINE SUCCESS
```

And:

```text
Image:
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

---

# Part 33 — Verify Docker Hub

Open:

```text
https://hub.docker.com/
```

Open:

```text
jenkins-demo-app
```

You should now see:

```text
10.0
```

---

# Part 34 — Pull the Jenkins-Built Image

To prove the image is actually in Docker Hub, remove the local image first.

Run:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

Check:

```bash
docker images | grep jenkins-demo-app
```

Now pull:

```bash
docker pull YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

Check:

```bash
docker images | grep jenkins-demo-app
```

You should see:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app   10.0
```

---

# Part 35 — Run the Image Pulled from Docker Hub

Run:

```bash
docker run --rm YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
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

# Part 36 — Test Another Image Version

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
11.0
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

Wait for the pipeline to finish.

---

# Part 37 — Verify Docker Hub Version 11.0

Open Docker Hub.

You should now see:

```text
10.0
11.0
```

The repository is now storing multiple application versions.

---

# Part 38 — Check Docker Images Locally

Run:

```bash
docker images | grep jenkins-demo-app
```

You may see:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app   10.0
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app   11.0
```

---

# Part 39 — Understand Docker Registry

Docker Hub is a container registry.

The architecture is:

```text
Jenkins
   |
   | docker build
   v
Docker Image
   |
   | docker login
   v
Docker Hub
   |
   | docker push
   v
Docker Registry
```

Later:

```text
Kubernetes
   |
   | docker pull
   v
Docker Hub
```

---

# Part 40 — Understand Image Naming

The full image name is:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

Breakdown:

```text
YOUR-DOCKERHUB-USERNAME
        |
        v
Docker Hub namespace

jenkins-demo-app
        |
        v
Repository

10.0
        |
        v
Tag
```

---

# Part 41 — Understand Docker Push

The command:

```bash
docker push YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

means:

```text
Take the local image
        |
        v
Authenticate to Docker Hub
        |
        v
Upload image layers
        |
        v
Store image in Docker Hub
```

---

# Part 42 — Understand Docker Pull

The command:

```bash
docker pull YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

means:

```text
Connect to Docker Hub
        |
        v
Find repository
        |
        v
Find tag 10.0
        |
        v
Download image
```

---

# Part 43 — Verify Jenkins Credentials Are Not in Git

Go to the application repository:

```bash
cd /c/project/jenkins-demo-app
```

Search the repository for common credential text:

```bash
grep -Rni "dckr_pat" . --exclude-dir=.git
```

Expected:

```text
```

There should be no PAT in your source files.

Search for:

```bash
grep -Rni "docker login" . --exclude-dir=.git
```

You may see your Jenkinsfile command, which is expected.

The token itself must not be present.

---

# Part 44 — Check Git Status

Run:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# Part 45 — Inspect Docker Images from Jenkins

Run:

```bash
docker exec jenkins docker images | grep jenkins-demo-app
```

You should see the images Jenkins built.

---

# Part 46 — Check Docker Hub Authentication from Jenkins

Do not manually print the credential.

Instead, use the Jenkins pipeline.

The important flow is:

```text
Jenkins Credentials
        |
        v
withCredentials
        |
        v
Docker Login
        |
        v
Docker Push
```

---

# Part 47 — Understand Why Credentials Belong in Jenkins

Bad:

```text
Jenkinsfile
    |
    +-- Docker username
    |
    +-- Docker password
```

Better:

```text
Jenkins
   |
   +-- Credentials
          |
          +-- dockerhub-credentials
                   |
                   v
              Jenkinsfile
```

The Jenkinsfile only references:

```text
dockerhub-credentials
```

and does not contain the secret itself. :contentReference[oaicite:9]{index=9}

---

# Part 48 — Test the Pipeline with Production

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
12.0
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

The pipeline should:

```text
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
Docker Login
   |
   v
Docker Push
```

---

# Part 49 — Verify Docker Hub Production Image

Open:

```text
Docker Hub
→ jenkins-demo-app
```

You should see:

```text
12.0
```

---

# Part 50 — Pull Production Image

Run:

```bash
docker pull YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:12.0
```

Run:

```bash
docker run --rm YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:12.0
```

---

# Part 51 — Check Build Information

In Jenkins, open the latest build.

Find:

```text
Build Artifacts
```

Open:

```text
build/build-info.txt
```

Expected format:

```text
Application=jenkins-demo-app
DockerImage=YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:12.0
Environment=prod
JenkinsBuild=<build-number>
GitCommit=<commit-hash>
GitBranch=main
```

---

# Part 52 — Understand the Complete Workflow

Your system is now:

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
    +---- Docker Login
    |
    +---- Docker Push
    |
    v
Docker Hub
    |
    v
Docker Image
```

---

# Part 53 — Real CI/CD Architecture So Far

You have now built:

```text
GitHub
   |
   | Source Code
   v
Jenkins
   |
   | Continuous Integration
   v
Docker Image
   |
   | Push
   v
Docker Hub
```

The next major step is:

```text
Docker Hub
    |
    v
Kubernetes
```

---

# Part 54 — Check Git History

Run:

```bash
cd /c/project/jenkins-demo-app
```

Run:

```bash
git log --oneline --max-count=10
```

You should see commits similar to:

```text
Add Docker Hub push to Jenkins pipeline
```

Your exact history will depend on previous labs.

---

# Part 55 — Check Git Status

Run:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# Part 56 — Save Lab 9 Documentation

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
touch lab-09-jenkins-dockerhub.md
```

Open:

```bash
code lab-09-jenkins-dockerhub.md
```

Paste this complete lab into the file.

Save.

---

# Part 57 — Check Git Status

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
new file: labs/lab-09-jenkins-dockerhub.md
```

---

# Part 58 — Add Lab 9

```bash
git add labs/lab-09-jenkins-dockerhub.md
```

Check:

```bash
git status
```

---

# Part 59 — Commit Lab 9

```bash
git commit -m "Add Jenkins Lab 9 Docker Hub integration"
```

---

# Part 60 — Push Lab 9

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

# Part 61 — Verify Repository Structure

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
    └── lab-09-jenkins-dockerhub.md
```

---

# Lab 9 Completion Checklist

```text
[ ] Docker Hub account verified
[ ] Docker Hub repository created
[ ] jenkins-demo-app repository created on Docker Hub
[ ] Docker Hub PAT created
[ ] Docker Hub PAT permissions configured for push
[ ] Local Docker login tested
[ ] Local Docker image created
[ ] Docker image manually pushed
[ ] Docker image manually pulled
[ ] Docker image tested
[ ] Jenkins Docker Hub credential created
[ ] Credential ID created
[ ] Credential ID is dockerhub-credentials
[ ] Docker Hub token not stored in Jenkinsfile
[ ] Jenkinsfile updated
[ ] Docker Login stage created
[ ] Docker Push stage created
[ ] Jenkins Docker image built
[ ] Jenkins Docker image tested
[ ] Jenkins logged into Docker Hub
[ ] Jenkins pushed image to Docker Hub
[ ] Docker Hub tag verified
[ ] Image pulled from Docker Hub
[ ] Image tested after pull
[ ] Multiple image versions pushed
[ ] Production image tested
[ ] Build artifact verified
[ ] Git status clean
[ ] Lab 9 documentation saved
[ ] Lab 9 committed
[ ] Lab 9 pushed to GitHub
```

---

# Lab 9 Cleanup

## Recommended

If you are continuing to Lab 10, keep:

```text
jenkins container
jenkins_home volume
jenkins network
```

Keep:

```text
jenkins-demo-app
```

Keep the Docker Hub repository:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

We will use the image in the Kubernetes lab.

---

# Remove Local Docker Images

List:

```bash
docker images | grep jenkins-demo-app
```

Remove a specific image:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:10.0
```

Remove another:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:11.0
```

Remove another:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:12.0
```

Only remove images you do not need.

---

# Remove Dangling Images

Check:

```bash
docker images --filter "dangling=true"
```

Remove unused dangling images:

```bash
docker image prune
```

---

# Revoke the Docker Hub Token

If you want to clean up the lab credentials, go to:

```text
Docker Hub
→ Account Settings
→ Personal access tokens
```

Find:

```text
jenkins-ci
```

Disable or delete the token.

Docker documents that PATs can be revoked/disabled and should be treated like passwords. :contentReference[oaicite:10]{index=10}

---

# Delete Jenkins Docker Hub Credential

From Jenkins:

```text
Manage Jenkins
→ Credentials
→ Global
```

Find:

```text
dockerhub-credentials
```

Delete it if you no longer need it.

---

# Complete Jenkins Cleanup

Only use these commands if you want to completely remove Jenkins.

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

Verify containers:

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
Docker Image Tags
Jenkins → Docker Hub
Credential Security
```

---

# Next Lab

## Lab 10 — Jenkins + Kubernetes

The next step is to connect everything we have built to Kubernetes.

The workflow will become:

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
    | image pull
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

We will use your existing **Minikube** environment and learn how Jenkins can deploy the Docker image to Kubernetes.

# End of Lab 9
