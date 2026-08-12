# Jenkins Zero to Hero — Lab 14

## Jenkins Shared Libraries

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
Lab 10 → Jenkins + Kubernetes
Lab 11 → Complete Jenkins CI/CD Pipeline
Lab 12 → Jenkins Credentials
Lab 13 → Jenkins Controller and Agents
```

In previous labs, we put most of the pipeline logic directly inside:

```text
Jenkinsfile
```

That works for one project.

But imagine you have:

```text
Project A
Project B
Project C
Project D
```

and all four projects need:

```text
Build
Test
Docker Build
Docker Push
Kubernetes Deploy
```

Copying the same Jenkinsfile logic into every repository creates duplication.

Shared Libraries solve this problem.

The architecture becomes:

```text
                   Jenkins Shared Library
                            |
                +-----------+-----------+
                |           |           |
                v           v           v
             Project A   Project B   Project C
                |           |           |
                v           v           v
            Jenkinsfile Jenkinsfile Jenkinsfile
```

The Jenkinsfiles become smaller because common pipeline logic is stored centrally.

---

# Lab Objective

By the end of this lab, you will understand:

```text
Jenkins Shared Libraries
vars/
src/
@Library
Global Variables
Reusable Pipeline Functions
Global Shared Library
Library Version
Library Repository
Library Import
Reusable Jenkins Functions
Centralized Pipeline Code
```

You will create:

```text
1. A GitHub Shared Library repository
2. A vars/ directory
3. Reusable Jenkins functions
4. A Jenkins Global Shared Library
5. A Jenkinsfile that imports the library
6. A pipeline that uses the shared functions
```

---

# Important

We will create a second GitHub repository:

```text
jenkins-shared-library
```

Your repositories will now look like:

```text
GitHub
│
├── jenkins-zero-to-hero
│
├── jenkins-demo-app
│
└── jenkins-shared-library
```

The purpose of the new repository is:

```text
Reusable Jenkins Pipeline Code
```

---

# Part 1 — Check Jenkins

Open Git Bash.

Run:

```bash
docker ps
```

You should see:

```text
jenkins
```

Check Jenkins:

```text
http://localhost:8081
```

---

# Part 2 — Check Docker Access From Jenkins

Run:

```bash
docker exec jenkins docker --version
```

You should see a Docker version.

---

# Part 3 — Check Kubernetes Access From Jenkins

Run:

```bash
docker exec jenkins kubectl get nodes
```

You should see your Minikube node or nodes.

---

# Part 4 — Create the Shared Library Repository on GitHub

Open GitHub.

Create a new repository:

```text
jenkins-shared-library
```

Use:

```text
Visibility:
Public
```

For this lab, do not initialize it with:

```text
README
.gitignore
License
```

We will create everything locally.

---

# Part 5 — Clone the Shared Library Repository

Open Git Bash.

Go to:

```bash
cd /c/project
```

Clone:

```bash
git clone https://github.com/YOUR-GITHUB-USERNAME/jenkins-shared-library.git
```

Enter:

```bash
cd jenkins-shared-library
```

Check:

```bash
pwd
```

Check:

```bash
ls
```

The repository should be empty.

---

# Part 6 — Create the Shared Library Structure

Create the main directories:

```bash
mkdir vars
```

Create the source directory:

```bash
mkdir src
```

Create a resources directory:

```bash
mkdir resources
```

Check:

```bash
ls
```

Expected:

```text
resources
src
vars
```

---

# Part 7 — Understand Shared Library Structure

The basic structure is:

```text
jenkins-shared-library/
│
├── vars/
│
├── src/
│
└── resources/
```

The important directories for this lab are:

```text
vars/
src/
```

---

# Part 8 — Understand `vars/`

The `vars/` directory contains reusable global pipeline steps.

For example:

```text
vars/buildApp.groovy
```

can be called from a Jenkinsfile as:

```groovy
buildApp()
```

Another example:

```text
vars/testApp.groovy
```

can be called as:

```groovy
testApp()
```

---

# Part 9 — Create a Build Function

Go to:

```bash
cd /c/project/jenkins-shared-library
```

Create:

```bash
touch vars/buildApp.groovy
```

Open:

```bash
code vars/buildApp.groovy
```

Paste:

```groovy
def call() {

    echo '======================================'
    echo 'SHARED LIBRARY: BUILD'
    echo '======================================'

    sh 'echo "Building application..."'

    sh 'test -f app.sh'

    sh 'test -f version.txt'

    echo 'Application build validation completed.'
}
```

Save.

---

# Part 10 — Understand `call()`

This:

```groovy
def call() {
}
```

allows the shared library function to be called simply as:

```groovy
buildApp()
```

without having to write:

```groovy
buildApp.call()
```

---

# Part 11 — Create a Test Function

Create:

```bash
touch vars/testApp.groovy
```

Open:

```bash
code vars/testApp.groovy
```

Paste:

```groovy
def call() {

    echo '======================================'
    echo 'SHARED LIBRARY: TEST'
    echo '======================================'

    sh 'test -f app.sh'

    sh 'test -f version.txt'

    sh 'grep -q "JENKINS DEMO APPLICATION" app.sh'

    sh 'grep -q "Application is running successfully." app.sh'

    echo 'Application tests passed.'
}
```

Save.

---

# Part 12 — Create a Docker Build Function

Create:

```bash
touch vars/dockerBuild.groovy
```

Open:

```bash
code vars/dockerBuild.groovy
```

Paste:

```groovy
def call(String imageName, String imageTag) {

    echo '======================================'
    echo 'SHARED LIBRARY: DOCKER BUILD'
    echo '======================================'

    echo "Image: ${imageName}:${imageTag}"

    sh "docker build -t ${imageName}:${imageTag} ."

    sh "docker image inspect ${imageName}:${imageTag} > /dev/null"

    echo 'Docker image created successfully.'
}
```

Save.

---

# Part 13 — Understand Function Parameters

This function:

```groovy
def call(String imageName, String imageTag)
```

expects two values.

For example:

```groovy
dockerBuild(
    'myuser/jenkins-demo-app',
    '1.0'
)
```

The result becomes:

```text
myuser/jenkins-demo-app:1.0
```

---

# Part 14 — Create a Docker Test Function

Create:

```bash
touch vars/dockerTest.groovy
```

Open:

```bash
code vars/dockerTest.groovy
```

Paste:

```groovy
def call(String imageName, String imageTag) {

    echo '======================================'
    echo 'SHARED LIBRARY: DOCKER TEST'
    echo '======================================'

    echo "Testing image: ${imageName}:${imageTag}"

    sh "docker run --rm ${imageName}:${imageTag}"

    echo 'Docker image test passed.'
}
```

Save.

---

# Part 15 — Create a Docker Push Function

Create:

```bash
touch vars/dockerPush.groovy
```

Open:

```bash
code vars/dockerPush.groovy
```

Paste:

```groovy
def call(String imageName, String imageTag, String credentialId) {

    echo '======================================'
    echo 'SHARED LIBRARY: DOCKER PUSH'
    echo '======================================'

    withCredentials([
        usernamePassword(
            credentialsId: credentialId,
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_TOKEN'
        )
    ]) {

        sh '''
            echo "$DOCKER_TOKEN" | docker login \
                --username "$DOCKER_USER" \
                --password-stdin
        '''

        sh "docker push ${imageName}:${imageTag}"
    }

    echo 'Docker image pushed successfully.'
}
```

Save.

---

# Part 16 — Create a Kubernetes Deploy Function

Create:

```bash
touch vars/k8sDeploy.groovy
```

Open:

```bash
code vars/k8sDeploy.groovy
```

Paste:

```groovy
def call(
    String namespace,
    String deploymentName,
    String containerName,
    String imageName,
    String imageTag
) {

    echo '======================================'
    echo 'SHARED LIBRARY: KUBERNETES DEPLOY'
    echo '======================================'

    echo "Namespace: ${namespace}"

    echo "Deployment: ${deploymentName}"

    echo "Image: ${imageName}:${imageTag}"

    sh """
        kubectl set image \
        deployment/${deploymentName} \
        ${containerName}=${imageName}:${imageTag} \
        -n ${namespace}
    """

    sh """
        kubectl rollout status \
        deployment/${deploymentName} \
        -n ${namespace} \
        --timeout=120s
    """

    echo 'Kubernetes deployment completed.'
}
```

Save.

---

# Part 17 — Create a Kubernetes Verify Function

Create:

```bash
touch vars/k8sVerify.groovy
```

Open:

```bash
code vars/k8sVerify.groovy
```

Paste:

```groovy
def call(String namespace, String deploymentName) {

    echo '======================================'
    echo 'SHARED LIBRARY: KUBERNETES VERIFY'
    echo '======================================'

    sh "kubectl get deployment ${deploymentName} -n ${namespace}"

    sh "kubectl get pods -n ${namespace} -l app=${deploymentName} -o wide"

    sh "kubectl rollout status deployment/${deploymentName} -n ${namespace} --timeout=120s"

    echo 'Kubernetes verification completed.'
}
```

Save.

---

# Part 18 — Check Shared Library Files

Run:

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

# Part 19 — Check Shared Library Structure

Run:

```bash
find . -maxdepth 2 -type f -print
```

You should see:

```text
./vars/buildApp.groovy
./vars/testApp.groovy
./vars/dockerBuild.groovy
./vars/dockerTest.groovy
./vars/dockerPush.groovy
./vars/k8sDeploy.groovy
./vars/k8sVerify.groovy
```

---

# Part 20 — Create a README

Create:

```bash
touch README.md
```

Open:

```bash
code README.md
```

Paste:

```markdown
# Jenkins Shared Library

Reusable Jenkins Pipeline functions.

## Functions

- buildApp()
- testApp()
- dockerBuild(imageName, imageTag)
- dockerTest(imageName, imageTag)
- dockerPush(imageName, imageTag, credentialId)
- k8sDeploy(namespace, deploymentName, containerName, imageName, imageTag)
- k8sVerify(namespace, deploymentName)
```

Save.

---

# Part 21 — Check Git Status

Run:

```bash
git status
```

You should see:

```text
README.md
vars/
```

---

# Part 22 — Add the Shared Library

Run:

```bash
git add .
```

Check:

```bash
git status
```

---

# Part 23 — Commit the Shared Library

Run:

```bash
git commit -m "Add Jenkins shared library functions"
```

---

# Part 24 — Push to GitHub

Run:

```bash
git push origin main
```

---

# Part 25 — Verify GitHub

Open:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-shared-library
```

You should see:

```text
README.md
resources/
src/
vars/
```

Inside:

```text
vars/
```

you should see:

```text
buildApp.groovy
testApp.groovy
dockerBuild.groovy
dockerTest.groovy
dockerPush.groovy
k8sDeploy.groovy
k8sVerify.groovy
```

---

# Part 26 — Configure the Jenkins Global Shared Library

Open Jenkins:

```text
http://localhost:8081
```

Go to:

```text
Manage Jenkins
```

Find:

```text
System
```

Depending on the Jenkins UI version, shared libraries may be under:

```text
Manage Jenkins
→ System
```

Scroll to:

```text
Global Pipeline Libraries
```

Click:

```text
Add
```

---

# Part 27 — Configure Library Name

For:

```text
Name
```

enter:

```text
jenkins-shared-library
```

This name will be used by Jenkins pipelines.

---

# Part 28 — Configure Default Version

For:

```text
Default version
```

enter:

```text
main
```

This tells Jenkins to use:

```text
main
```

from the Git repository unless a pipeline specifies another version.

---

# Part 29 — Configure Retrieval Method

For:

```text
Retrieval method
```

select:

```text
Modern SCM
```

For:

```text
Source Code Management
```

select:

```text
Git
```

---

# Part 30 — Enter Repository URL

Repository URL:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-shared-library.git
```

For a public repository:

```text
Credentials:
none
```

Save the Jenkins configuration.

---

# Part 31 — Understand Global Shared Library

The configuration is now:

```text
Jenkins
   |
   v
Global Pipeline Libraries
   |
   v
jenkins-shared-library
   |
   v
GitHub Repository
```

Jenkins can now load functions from:

```text
vars/
```

---

# Part 32 — Create a Shared Library Test Job

Go to:

```text
Jenkins Dashboard
```

Click:

```text
New Item
```

Enter:

```text
lab-14-shared-library-test
```

Select:

```text
Pipeline
```

Click:

```text
OK
```

---

# Part 33 — Import the Shared Library

Go to:

```text
Pipeline
```

Use:

```text
Pipeline script
```

Paste:

```groovy
@Library('jenkins-shared-library') _

pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                buildApp()
            }
        }

        stage('Test') {

            steps {

                testApp()
            }
        }
    }

    post {

        success {
            echo 'Shared library pipeline completed successfully.'
        }

        failure {
            echo 'Shared library pipeline failed.'
        }
    }
}
```

Click:

```text
Save
```

---

# Part 34 — Understand `@Library`

This line:

```groovy
@Library('jenkins-shared-library') _
```

tells Jenkins:

```text
Load the shared library called:
jenkins-shared-library
```

Then Jenkins can use:

```groovy
buildApp()
```

and:

```groovy
testApp()
```

---

# Part 35 — Run the Shared Library Pipeline

Click:

```text
Build Now
```

Open:

```text
Console Output
```

You should see:

```text
SHARED LIBRARY: BUILD
```

Then:

```text
SHARED LIBRARY: TEST
```

---

# Part 36 — Verify the Shared Library Worked

The important part is that the Jenkinsfile did not contain:

```text
Build shell commands
Test shell commands
```

Instead it contained:

```groovy
buildApp()
testApp()
```

The actual logic came from:

```text
jenkins-shared-library
```

---

# Part 37 — Use Docker Shared Library Functions

Now create another test job:

```text
lab-14-docker-library-test
```

Select:

```text
Pipeline
```

---

# Part 38 — Configure Docker Library Test

Use:

```groovy
@Library('jenkins-shared-library') _

pipeline {

    agent any

    environment {
        IMAGE_NAME = 'YOUR-DOCKERHUB-USERNAME/jenkins-demo-app'
        IMAGE_TAG = '50.0'
    }

    stages {

        stage('Checkout') {

            steps {

                checkout scm
            }
        }

        stage('Docker Build') {

            steps {

                dockerBuild(
                    "${IMAGE_NAME}",
                    "${IMAGE_TAG}"
                )
            }
        }

        stage('Docker Test') {

            steps {

                dockerTest(
                    "${IMAGE_NAME}",
                    "${IMAGE_TAG}"
                )
            }
        }
    }
}
```

Replace:

```text
YOUR-DOCKERHUB-USERNAME
```

with your Docker Hub username.

Save.

---

# Part 39 — Important Repository Requirement

The test job above uses:

```groovy
checkout scm
```

So the job needs access to the application repository.

A simpler learning option is to configure:

```text
Pipeline script from SCM
```

and use:

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

For this shared-library test, however, the main thing we are testing is the library function.

---

# Part 40 — Easier Docker Library Test

For a simple test, use a pipeline that checks Docker only.

Create:

```text
lab-14-docker-library-simple
```

Use:

```groovy
@Library('jenkins-shared-library') _

pipeline {

    agent any

    stages {

        stage('Docker Check') {

            steps {

                sh 'docker --version'

                sh 'docker ps'
            }
        }
    }
}
```

Save.

This confirms the Jenkins environment can access Docker.

---

# Part 41 — Use Docker Build Function in the Real Application Pipeline

Now we will use the shared library with the real application repository.

Go to:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

At the top add:

```groovy
@Library('jenkins-shared-library') _
```

---

# Part 42 — Simplify the Jenkinsfile

The shared-library pipeline can now look like:

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

                checkout scm
            }
        }

        stage('Build') {

            steps {

                buildApp()
            }
        }

        stage('Test') {

            steps {

                testApp()
            }
        }

        stage('Docker Build') {

            steps {

                dockerBuild(
                    "${IMAGE}",
                    "${APP_VERSION}"
                )
            }
        }

        stage('Docker Test') {

            steps {

                dockerTest(
                    "${IMAGE}",
                    "${APP_VERSION}"
                )
            }
        }

        stage('Docker Push') {

            steps {

                dockerPush(
                    "${IMAGE}",
                    "${APP_VERSION}",
                    'dockerhub-credentials'
                )
            }
        }

        stage('Kubernetes Deploy') {

            steps {

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

                k8sVerify(
                    "${K8S_NAMESPACE}",
                    "${K8S_DEPLOYMENT}"
                )
            }
        }
    }

    post {

        success {

            echo '======================================'

            echo 'SHARED LIBRARY CI/CD SUCCESS'

            echo '======================================'

            echo "Image: ${IMAGE}:${params.APP_VERSION}"

            echo "Environment: ${params.ENVIRONMENT}"
        }

        failure {

            echo 'Shared library CI/CD pipeline failed.'
        }

        always {

            echo 'Shared library pipeline finished.'
        }
    }
}
```

Replace:

```text
YOUR-DOCKERHUB-USERNAME
```

with your Docker Hub username.

---

# Part 43 — Compare Old and New Pipeline

Before:

```text
Large Jenkinsfile
    |
    +---- Build commands
    +---- Test commands
    +---- Docker commands
    +---- Kubernetes commands
```

After:

```text
Small Jenkinsfile
    |
    +---- buildApp()
    +---- testApp()
    +---- dockerBuild()
    +---- dockerTest()
    +---- dockerPush()
    +---- k8sDeploy()
    +---- k8sVerify()
```

The logic is now centralized.

---

# Part 44 — Check Jenkinsfile

Run:

```bash
grep -n "Library" Jenkinsfile
```

Expected:

```text
@Library('jenkins-shared-library') _
```

Check shared functions:

```bash
grep -nE "buildApp|testApp|dockerBuild|dockerTest|dockerPush|k8sDeploy|k8sVerify" Jenkinsfile
```

You should see the shared library functions.

---

# Part 45 — Commit the Shared Library Jenkinsfile

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
git commit -m "Use Jenkins shared library in CI/CD pipeline"
```

---

# Part 46 — Push the Application Pipeline

Run:

```bash
git push origin main
```

---

# Part 47 — Run the Real Pipeline

Open:

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
50.0
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

# Part 48 — Check Pipeline Output

Open:

```text
Console Output
```

You should see:

```text
SHARED LIBRARY: BUILD
```

Then:

```text
SHARED LIBRARY: TEST
```

Then:

```text
SHARED LIBRARY: DOCKER BUILD
```

Then:

```text
SHARED LIBRARY: DOCKER TEST
```

Then:

```text
SHARED LIBRARY: DOCKER PUSH
```

Then:

```text
SHARED LIBRARY: KUBERNETES DEPLOY
```

Then:

```text
SHARED LIBRARY: KUBERNETES VERIFY
```

---

# Part 49 — Verify Docker Hub

Open Docker Hub.

Check:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

You should see:

```text
50.0
```

---

# Part 50 — Verify Kubernetes

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo
```

Check the image:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:50.0
```

---

# Part 51 — Verify Kubernetes Pods

Run:

```bash
kubectl get pods -n jenkins-demo
```

They should be:

```text
Running
```

Check rollout:

```bash
kubectl rollout status deployment/jenkins-demo-app -n jenkins-demo
```

---

# Part 52 — Understand the Shared Library Architecture

The complete architecture is now:

```text
                   GitHub
                     |
                     v
          Jenkins Shared Library
                     |
        +------------+------------+
        |            |            |
        v            v            v
    buildApp()   testApp()   dockerBuild()
        |            |            |
        +------------+------------+
                     |
                     v
              Jenkinsfile
                     |
                     v
                  Jenkins
                     |
        +------------+------------+
        |                         |
        v                         v
    Docker Hub                Kubernetes
```

---

# Part 53 — Understand `vars/`

Our library contains:

```text
vars/
├── buildApp.groovy
├── testApp.groovy
├── dockerBuild.groovy
├── dockerTest.groovy
├── dockerPush.groovy
├── k8sDeploy.groovy
└── k8sVerify.groovy
```

Each file provides a reusable global pipeline step.

---

# Part 54 — Understand `src/`

The `src/` directory is used for more structured Groovy classes and reusable code.

Example structure:

```text
src/
└── org/
    └── example/
        └── pipeline/
            └── BuildHelper.groovy
```

We are not using `src/` heavily in this first Shared Library lab.

---

# Part 55 — Understand `resources/`

The:

```text
resources/
```

directory can contain non-Groovy resources used by libraries.

For example:

```text
resources/
└── templates/
    └── config.txt
```

We are not using it in this lab.

---

# Part 56 — Check Shared Library Git Status

Go to:

```bash
cd /c/project/jenkins-shared-library
```

Run:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

If you made changes after the initial push:

```bash
git add .
```

Commit:

```bash
git commit -m "Update Jenkins shared library functions"
```

Push:

```bash
git push origin main
```

---

# Part 57 — Make a Shared Library Change

We will prove that changing the library changes the pipeline behavior.

Open:

```bash
code vars/buildApp.groovy
```

Change:

```groovy
echo 'SHARED LIBRARY: BUILD'
```

to:

```groovy
echo 'SHARED LIBRARY: BUILD — VERSION 2'
```

Save.

---

# Part 58 — Commit the Library Change

Run:

```bash
git add vars/buildApp.groovy
```

Commit:

```bash
git commit -m "Update shared build function"
```

Push:

```bash
git push origin main
```

---

# Part 59 — Run the Application Pipeline Again

Open:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
51.0
```

Environment:

```text
dev
```

Click:

```text
Build
```

---

# Part 60 — Verify Shared Library Update

Open:

```text
Console Output
```

You should now see:

```text
SHARED LIBRARY: BUILD — VERSION 2
```

This proves that the Jenkins pipeline is loading the shared library from Git.

---

# Part 61 — Understand Library Reuse

Now imagine you have:

```text
Project A
Project B
Project C
```

All three can use:

```text
buildApp()
testApp()
dockerBuild()
dockerPush()
k8sDeploy()
```

Changing:

```text
dockerPush.groovy
```

can update the common behavior used by multiple pipelines.

This is why Shared Libraries are useful for organizations with many Jenkins projects.

---

# Part 62 — Understand Library Versioning

A shared library can have versions.

For example:

```text
main
v1.0
v1.1
v2.0
```

A pipeline can use a specific branch or version.

Example:

```groovy
@Library('jenkins-shared-library@main') _
```

Or:

```groovy
@Library('jenkins-shared-library@v1.0') _
```

---

# Part 63 — Understand Why Versioning Matters

Without versioning:

```text
Project A
Project B
Project C
       |
       v
Latest Library
```

A change to the library can affect every project.

With versioning:

```text
Project A → Library v1.0
Project B → Library v1.0
Project C → Library v2.0
```

Teams can upgrade deliberately.

---

# Part 64 — Use a Specific Library Version

If you have a branch called:

```text
main
```

you can explicitly reference it:

```groovy
@Library('jenkins-shared-library@main') _
```

For the current lab, using:

```groovy
@Library('jenkins-shared-library') _
```

uses the configured default version.

---

# Part 65 — Check the Full Shared Library Repository

Run:

```bash
cd /c/project/jenkins-shared-library
```

Run:

```bash
find . -maxdepth 2 -type f -print
```

You should see:

```text
./README.md
./vars/buildApp.groovy
./vars/testApp.groovy
./vars/dockerBuild.groovy
./vars/dockerTest.groovy
./vars/dockerPush.groovy
./vars/k8sDeploy.groovy
./vars/k8sVerify.groovy
```

---

# Part 66 — Check the Application Repository

Run:

```bash
cd /c/project/jenkins-demo-app
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

# Part 67 — Check the Application Jenkinsfile Size

Run:

```bash
wc -l Jenkinsfile
```

The Jenkinsfile should be significantly simpler than the large pipeline we used previously.

---

# Part 68 — Why Shared Libraries Matter

Without shared libraries:

```text
Project A
   |
   +---- 150 lines

Project B
   |
   +---- 150 lines

Project C
   |
   +---- 150 lines
```

With shared libraries:

```text
Shared Library
   |
   +---- Build
   +---- Test
   +---- Docker
   +---- Kubernetes
          |
          v
       Projects
```

This reduces duplicated pipeline logic.

---

# Part 69 — Shared Library Best Practice

Keep:

```text
Jenkinsfile
```

focused on:

```text
Pipeline structure
Project-specific configuration
Parameters
Environment
Stage ordering
```

Keep reusable logic inside:

```text
Shared Library
```

For example:

```text
dockerBuild()
dockerPush()
k8sDeploy()
```

---

# Part 70 — Check Git History of Shared Library

Run:

```bash
cd /c/project/jenkins-shared-library
```

Then:

```bash
git log --oneline --max-count=10
```

You should see commits such as:

```text
Update shared build function
Add Jenkins shared library functions
```

Your exact history may differ.

---

# Part 71 — Check Git History of Application

Run:

```bash
cd /c/project/jenkins-demo-app
```

Then:

```bash
git log --oneline --max-count=10
```

You should see the Jenkinsfile shared-library commit.

---

# Part 72 — Save Lab 14 Documentation

Go to:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go to:

```bash
cd labs
```

Create:

```bash
touch lab-14-jenkins-shared-libraries.md
```

Open:

```bash
code lab-14-jenkins-shared-libraries.md
```

Paste this complete lab into the file.

Save.

---

# Part 73 — Check Git Status

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
new file: labs/lab-14-jenkins-shared-libraries.md
```

---

# Part 74 — Add Lab 14

```bash
git add labs/lab-14-jenkins-shared-libraries.md
```

Check:

```bash
git status
```

---

# Part 75 — Commit Lab 14

```bash
git commit -m "Add Jenkins Lab 14 shared libraries"
```

---

# Part 76 — Push Lab 14

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

# Part 77 — Verify Repository Structure

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
    ├── lab-11-complete-cicd-pipeline.md
    │
    ├── lab-12-jenkins-credentials.md
    │
    ├── lab-13-jenkins-agents.md
    │
    └── lab-14-jenkins-shared-libraries.md
```

---

# Lab 14 Completion Checklist

```text
[ ] Shared library GitHub repository created
[ ] Shared library repository cloned
[ ] vars directory created
[ ] src directory created
[ ] resources directory created
[ ] buildApp function created
[ ] testApp function created
[ ] dockerBuild function created
[ ] dockerTest function created
[ ] dockerPush function created
[ ] k8sDeploy function created
[ ] k8sVerify function created
[ ] Shared library committed
[ ] Shared library pushed to GitHub
[ ] Global Pipeline Library configured
[ ] Library name configured
[ ] Library Git repository configured
[ ] Library default branch configured
[ ] @Library understood
[ ] Shared library test job created
[ ] buildApp tested
[ ] testApp tested
[ ] Docker shared functions tested
[ ] Kubernetes shared functions tested
[ ] Real application Jenkinsfile uses shared library
[ ] Docker image built using shared library
[ ] Docker image pushed using shared library
[ ] Kubernetes deployment performed using shared library
[ ] Shared library change tested
[ ] Library versioning understood
[ ] Git history checked
[ ] Lab 14 documentation saved
[ ] Lab 14 committed
[ ] Lab 14 pushed to GitHub
```

---

# Lab 14 Cleanup

## Delete Test Jobs

From Jenkins:

```text
Dashboard
→ lab-14-shared-library-test
→ Configure
→ Delete Project
```

Delete:

```text
lab-14-docker-library-test
```

Delete:

```text
lab-14-docker-library-simple
```

Keep:

```text
lab-05-pipeline
```

if you want to continue using the shared-library CI/CD pipeline.

---

# Remove Test Docker Images

Check:

```bash
docker images | grep jenkins-demo-app
```

Remove a test image if needed:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:50.0
```

Remove another:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:51.0
```

Only remove images you no longer need.

---

# Clean Kubernetes Application

If you want to remove the Kubernetes deployment:

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

To remove everything:

```bash
kubectl delete namespace jenkins-demo
```

Verify:

```bash
kubectl get namespaces
```

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
Docker
Docker Hub
Kubernetes
Automated Deployment
Failure Handling
Rollback
```

Lab 12:

```text
Jenkins Credentials
Secret Text
Username + Password
SSH Keys
Credential IDs
withCredentials
Credentials Binding
Secret Masking
GitHub Tokens
Docker Hub Tokens
Secret Management
```

Lab 13:

```text
Jenkins Controller
Jenkins Agent
Node
Executor
Label
Agent Workspace
Docker Agent
Distributed Builds
Pipeline Agent
Stage Agent
```

Lab 14:

```text
Jenkins Shared Libraries
vars/
src/
resources/
@Library
Global Pipeline Libraries
Reusable Pipeline Functions
Library Versioning
Centralized Pipeline Logic
```

---

# Next Lab

## Lab 15 — Jenkins Security

In the next lab, we will secure Jenkins.

We will learn:

```text
Jenkins Users
Authentication
Authorization
Roles
Permissions
Matrix Authorization
Credentials Security
CSRF Protection
Agent Security
Security Best Practices
```

The goal is to move from:

```text
Jenkins works
```

to:

```text
Jenkins is controlled and secured
```

# End of Lab 14
