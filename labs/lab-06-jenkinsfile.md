# Jenkins Zero to Hero — Lab 6

## Jenkinsfile Deep Dive

This lab continues from:

```text
Lab 1 → Jenkins Installation
Lab 2 → Jobs, Workspace and Parameters
Lab 3 → Jenkins + Git + GitHub
Lab 4 → GitHub Webhooks
Lab 5 → Jenkins Pipeline and Jenkinsfile
```

In Lab 5, you created your first Jenkins Pipeline.

In this lab, we will go deeper into the Jenkinsfile and learn how to control a real CI pipeline.

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
    v
Jenkinsfile
    |
    +---- Checkout
    |
    +---- Validate
    |
    +---- Build
    |
    +---- Test
    |
    +---- Package
    |
    +---- Archive
    |
    v
Success
```

---

# Lab Objective

By the end of this lab, you will understand:

```text
pipeline
agent
environment
parameters
options
stages
stage
steps
script
when
post
sh
echo
currentBuild
```

You will also build a more realistic CI pipeline.

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

If Jenkins is stopped:

```bash
docker start jenkins
```

Check again:

```bash
docker ps
```

Open:

```text
http://localhost:8081
```

---

# Part 1 — Go to the Application Repository

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

You should see:

```text
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

# Part 3 — Check Git Remote

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

# Part 4 — Create a Backup of the Current Jenkinsfile

Before changing the Jenkinsfile, make a backup.

Run:

```bash
cp Jenkinsfile Jenkinsfile.lab5-backup
```

Check:

```bash
ls
```

You should now see:

```text
Jenkinsfile
Jenkinsfile.lab5-backup
README.md
app.sh
version.txt
```

---

# Part 5 — Understand the Pipeline Structure

A Jenkins Declarative Pipeline has a structure similar to:

```groovy
pipeline {

    agent any

    environment {
    }

    parameters {
    }

    options {
    }

    stages {

        stage('Example') {
            steps {
            }
        }

    }

    post {
    }
}
```

Each section has a specific purpose.

---

# Part 6 — Understand pipeline

The first block is:

```groovy
pipeline {
}
```

This tells Jenkins:

```text
This file contains a Jenkins Declarative Pipeline.
```

Everything belongs inside this block.

---

# Part 7 — Understand agent

We previously used:

```groovy
agent any
```

This means Jenkins can execute the pipeline on any available agent.

For this lab, use:

```groovy
agent any
```

---

# Part 8 — Understand environment

The `environment` section creates variables available to the pipeline.

Example:

```groovy
environment {
    APP_NAME = 'jenkins-demo-app'
    APP_OWNER = 'devops-team'
}
```

These values can then be used inside stages.

---

# Part 9 — Understand parameters

Parameters allow the person starting the build to provide values.

Example:

```groovy
parameters {
    string(
        name: 'APP_VERSION',
        defaultValue: '1.0',
        description: 'Application version'
    )
}
```

The value can be accessed using:

```groovy
params.APP_VERSION
```

---

# Part 10 — Understand options

The `options` section controls pipeline behavior.

We will use:

```groovy
options {
    timestamps()
    disableConcurrentBuilds()
}
```

This does two things.

```text
timestamps()
```

adds timestamps to console output.

```text
disableConcurrentBuilds()
```

prevents two copies of the same pipeline from running at the same time.

---

# Part 11 — Create the New Jenkinsfile

Open:

```bash
code Jenkinsfile
```

Replace the existing Jenkinsfile with:

```groovy
pipeline {

    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
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
        BUILD_OUTPUT = 'build'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '======================================'
                echo 'CHECKOUT STAGE'
                echo '======================================'

                checkout scm
            }
        }

        stage('Validate') {
            steps {
                echo '======================================'
                echo 'VALIDATE STAGE'
                echo '======================================'

                sh 'test -f app.sh'
                sh 'test -f version.txt'
                sh 'test -f Jenkinsfile'

                echo 'Required files are present.'
            }
        }

        stage('Build') {
            steps {
                echo '======================================'
                echo 'BUILD STAGE'
                echo '======================================'

                echo "Application: ${APP_NAME}"
                echo "Version: ${params.APP_VERSION}"
                echo "Environment: ${params.ENVIRONMENT}"

                sh 'echo "Building application..."'

                sh 'chmod +x app.sh'

                sh './app.sh'
            }
        }

        stage('Test') {
            steps {
                echo '======================================'
                echo 'TEST STAGE'
                echo '======================================'

                sh 'test -s app.sh'
                sh 'test -s version.txt'

                sh 'grep -q "JENKINS DEMO APPLICATION" app.sh'

                echo 'Application tests passed.'
            }
        }

        stage('Package') {
            steps {
                echo '======================================'
                echo 'PACKAGE STAGE'
                echo '======================================'

                sh 'rm -rf "$BUILD_OUTPUT"'
                sh 'mkdir -p "$BUILD_OUTPUT"'

                sh 'cp app.sh "$BUILD_OUTPUT"/'
                sh 'cp version.txt "$BUILD_OUTPUT"/'

                sh 'echo "Application=${APP_NAME}" > "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "Version=${APP_VERSION}" >> "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "Environment=${ENVIRONMENT}" >> "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "BuildNumber=${BUILD_NUMBER}" >> "$BUILD_OUTPUT"/build-info.txt'

                sh 'cat "$BUILD_OUTPUT"/build-info.txt'
            }
        }

        stage('Archive') {
            steps {
                echo '======================================'
                echo 'ARCHIVE STAGE'
                echo '======================================'

                archiveArtifacts artifacts: 'build/**', fingerprint: true

                echo 'Build artifacts archived.'
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'PIPELINE SUCCESS'
            echo '======================================'

            echo "Application: ${APP_NAME}"
            echo "Version: ${params.APP_VERSION}"
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

Save the file.

---

# Part 12 — Check the Jenkinsfile

Run:

```bash
cat Jenkinsfile
```

Check that the file contains:

```text
Checkout
Validate
Build
Test
Package
Archive
```

---

# Part 13 — Validate the Application Locally

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
test -f Jenkinsfile && echo "Jenkinsfile exists"
```

Expected:

```text
Jenkinsfile exists
```

---

# Part 14 — Test the Application

Run:

```bash
chmod +x app.sh
```

Then:

```bash
./app.sh
```

Expected output will look similar to:

```text
======================================
JENKINS DEMO APPLICATION
======================================

Application: Jenkins Demo App
Environment: Jenkins CI
Version: 4.0

Application is running successfully.

======================================
```

Your version may be different.

---

# Part 15 — Check Application Version

Run:

```bash
cat version.txt
```

For example:

```text
4.0
```

---

# Part 16 — Commit the New Jenkinsfile

Check:

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
git commit -m "Build advanced Jenkins pipeline"
```

---

# Part 17 — Push the New Jenkinsfile

Run:

```bash
git push origin main
```

---

# Part 18 — Open Jenkins

Open:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

Because the job is configured as:

```text
Pipeline script from SCM
```

Jenkins will read the Jenkinsfile from GitHub.

---

# Part 19 — Run with Parameters

Click:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
1.0
```

Select:

```text
ENVIRONMENT:
dev
```

Click:

```text
Build
```

---

# Part 20 — Check Pipeline Stages

Open the new build.

You should see stages similar to:

```text
Checkout
Validate
Build
Test
Package
Archive
```

---

# Part 21 — Check Console Output

Open:

```text
Console Output
```

You should see timestamps because we enabled:

```groovy
timestamps()
```

The output should show:

```text
CHECKOUT STAGE
VALIDATE STAGE
BUILD STAGE
TEST STAGE
PACKAGE STAGE
ARCHIVE STAGE
```

---

# Part 22 — Check the Build Output

The pipeline should create:

```text
build/
```

Inside:

```text
build/
├── app.sh
├── version.txt
└── build-info.txt
```

---

# Part 23 — Inspect the Build Directory

From Git Bash:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline/build"
```

Expected:

```text
app.sh
version.txt
build-info.txt
```

---

# Part 24 — Read Build Information

Run:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-05-pipeline/build/build-info.txt"
```

Expected:

```text
Application=jenkins-demo-app
Version=1.0
Environment=dev
BuildNumber=<build-number>
```

The build number will be different.

---

# Part 25 — Check Archived Artifact

Open the Jenkins build.

Look for:

```text
Build Artifacts
```

You should see:

```text
build/
```

Open the artifact.

You should find:

```text
app.sh
version.txt
build-info.txt
```

---

# Part 26 — Test the Test Environment

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
2.0
```

Select:

```text
ENVIRONMENT:
test
```

Click:

```text
Build
```

Check:

```text
Console Output
```

You should see:

```text
Application: jenkins-demo-app
Version: 2.0
Environment: test
```

---

# Part 27 — Test the Production Environment

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
3.0
```

Select:

```text
ENVIRONMENT:
prod
```

Click:

```text
Build
```

Check:

```text
Console Output
```

Expected:

```text
Application: jenkins-demo-app
Version: 3.0
Environment: prod
```

---

# Part 28 — Understand `when`

The `when` condition allows a stage to run only when a condition is true.

We will add a production-only stage.

Open:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

Add this stage after the `Archive` stage:

```groovy
        stage('Production Check') {
            when {
                expression {
                    params.ENVIRONMENT == 'prod'
                }
            }

            steps {
                echo 'Production environment selected.'
                echo 'Running production checks...'

                sh 'echo "Production checks completed."'
            }
        }
```

Save.

---

# Part 29 — Commit the `when` Stage

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add production conditional stage"
```

Push:

```bash
git push origin main
```

---

# Part 30 — Test `when` with Development

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
4.0
```

Select:

```text
ENVIRONMENT:
dev
```

Click:

```text
Build
```

The:

```text
Production Check
```

stage should be skipped.

---

# Part 31 — Test `when` with Production

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
4.0
```

Select:

```text
ENVIRONMENT:
prod
```

Click:

```text
Build
```

Now:

```text
Production Check
```

should run.

Console output should include:

```text
Production environment selected.
Running production checks...
Production checks completed.
```

---

# Part 32 — Understand `script`

Declarative Pipeline normally expects structured syntax.

The `script` block allows more advanced Groovy logic.

We will create a simple example.

Open:

```bash
code Jenkinsfile
```

Inside the `Build` stage, replace:

```groovy
echo "Application: ${APP_NAME}"
echo "Version: ${params.APP_VERSION}"
echo "Environment: ${params.ENVIRONMENT}"
```

with:

```groovy
script {
    def fullVersion = "${params.APP_VERSION}-${BUILD_NUMBER}"

    echo "Application: ${APP_NAME}"
    echo "Version: ${params.APP_VERSION}"
    echo "Environment: ${params.ENVIRONMENT}"
    echo "Build Version: ${fullVersion}"
}
```

Save.

---

# Part 33 — Understand the `script` Example

This line:

```groovy
def fullVersion = "${params.APP_VERSION}-${BUILD_NUMBER}"
```

creates a new value.

For example:

```text
APP_VERSION = 4.0
BUILD_NUMBER = 10
```

The result becomes:

```text
4.0-10
```

This can later become a Docker image tag.

For example:

```text
jenkins-demo-app:4.0-10
```

---

# Part 34 — Commit the `script` Change

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add script block to Jenkins pipeline"
```

Push:

```bash
git push origin main
```

---

# Part 35 — Run the Pipeline

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
5.0
```

Select:

```text
ENVIRONMENT:
dev
```

Click:

```text
Build
```

Check:

```text
Console Output
```

You should see something similar to:

```text
Application: jenkins-demo-app
Version: 5.0
Environment: dev
Build Version: 5.0-<build-number>
```

---

# Part 36 — Understand `post`

We already have:

```groovy
post {

    success {
        echo 'Pipeline completed successfully.'
    }

    failure {
        echo 'Pipeline failed.'
    }

    always {
        echo 'Pipeline execution finished.'
    }
}
```

This means:

```text
success
    ↓
Runs only when pipeline succeeds

failure
    ↓
Runs only when pipeline fails

always
    ↓
Runs regardless of success or failure
```

---

# Part 37 — Intentionally Test a Failure

We will create a temporary failure.

Open:

```bash
code Jenkinsfile
```

Inside the `Validate` stage, add:

```groovy
sh 'test -f file-that-does-not-exist.txt'
```

The stage should look similar to:

```groovy
        stage('Validate') {
            steps {
                echo '======================================'
                echo 'VALIDATE STAGE'
                echo '======================================'

                sh 'test -f app.sh'
                sh 'test -f version.txt'
                sh 'test -f Jenkinsfile'
                sh 'test -f file-that-does-not-exist.txt'

                echo 'Required files are present.'
            }
        }
```

Save.

---

# Part 38 — Commit the Intentional Failure

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Test Jenkins post failure handling"
```

Push:

```bash
git push origin main
```

---

# Part 39 — Run the Failed Pipeline

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
6.0
```

Select:

```text
ENVIRONMENT:
dev
```

Click:

```text
Build
```

The pipeline should fail in:

```text
Validate
```

---

# Part 40 — Check the Failure

Open:

```text
Console Output
```

You should see a failure related to:

```text
file-that-does-not-exist.txt
```

The pipeline should show:

```text
PIPELINE FAILED
PIPELINE FINISHED
```

This demonstrates the `post` section.

---

# Part 41 — Fix the Failure

Open:

```bash
code Jenkinsfile
```

Remove:

```groovy
sh 'test -f file-that-does-not-exist.txt'
```

Save.

---

# Part 42 — Commit the Fix

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Fix Jenkins validation stage"
```

Push:

```bash
git push origin main
```

---

# Part 43 — Run the Fixed Pipeline

Open Jenkins:

```text
http://localhost:8081
```

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
6.1
```

Select:

```text
ENVIRONMENT:
dev
```

Click:

```text
Build
```

The pipeline should succeed.

---

# Part 44 — Test the Production Conditional Stage

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
7.0
```

Select:

```text
ENVIRONMENT:
prod
```

Click:

```text
Build
```

You should see:

```text
Production Check
```

execute.

---

# Part 45 — Check the Complete Stage Flow

Your pipeline now looks like:

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
Package
    |
    v
Archive
    |
    v
Production Check
    |
    v
Success
```

When the environment is not `prod`:

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
Package
    |
    v
Archive
    |
    v
Production Check
   SKIPPED
    |
    v
Success
```

---

# Part 46 — Check Jenkins Workspace

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline"
```

Check build:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline/build"
```

Read:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-05-pipeline/build/build-info.txt"
```

---

# Part 47 — Check Git History

Go to:

```bash
cd /c/project/jenkins-demo-app
```

Run:

```bash
git log --oneline --max-count=10
```

You should see commits related to:

```text
Build advanced Jenkins pipeline
Add production conditional stage
Add script block to Jenkins pipeline
Test Jenkins post failure handling
Fix Jenkins validation stage
```

Your exact commit hashes will be different.

---

# Part 48 — Check Git Status

Run:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# Part 49 — Understand the Final Jenkinsfile

The final structure is:

```text
pipeline
│
├── agent
│
├── options
│
├── parameters
│
├── environment
│
├── stages
│   │
│   ├── Checkout
│   ├── Validate
│   ├── Build
│   ├── Test
│   ├── Package
│   ├── Archive
│   └── Production Check
│
└── post
    ├── success
    ├── failure
    └── always
```

---

# Part 50 — Jenkinsfile Keywords Learned

```text
pipeline
```

Defines the complete pipeline.

```text
agent
```

Defines where the pipeline executes.

```text
options
```

Controls pipeline behavior.

```text
parameters
```

Accepts user-provided build values.

```text
environment
```

Defines environment variables.

```text
stages
```

Contains all stages.

```text
stage
```

Defines one logical part of the pipeline.

```text
steps
```

Contains commands executed by a stage.

```text
script
```

Allows more advanced Groovy logic.

```text
when
```

Conditionally runs a stage.

```text
post
```

Runs actions after the pipeline completes.

---

# Part 51 — Jenkins Pipeline Architecture

The pipeline now resembles a real CI system:

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
    v
Jenkinsfile
    |
    +---- Checkout
    |
    +---- Validate
    |
    +---- Build
    |
    +---- Test
    |
    +---- Package
    |
    +---- Archive
    |
    +---- Conditional Deploy Check
    |
    v
Build Artifact
```

---

# Part 52 — Save Lab 6 Documentation

Go to your Jenkins documentation repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go to the labs folder:

```bash
cd labs
```

Create the file:

```bash
touch lab-06-jenkinsfile-deep-dive.md
```

Open it:

```bash
code lab-06-jenkinsfile-deep-dive.md
```

Paste this entire lab into the file.

Save it.

---

# Part 53 — Check Git Status

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
new file: labs/lab-06-jenkinsfile-deep-dive.md
```

---

# Part 54 — Add Lab 6

```bash
git add labs/lab-06-jenkinsfile-deep-dive.md
```

Check:

```bash
git status
```

---

# Part 55 — Commit Lab 6

```bash
git commit -m "Add Jenkins Lab 6 Jenkinsfile deep dive"
```

---

# Part 56 — Push Lab 6

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

# Part 57 — Verify Repository Structure

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
    └── lab-06-jenkinsfile-deep-dive.md
```

---

# Lab 6 Completion Checklist

```text
[ ] Jenkins is running
[ ] Jenkinsfile backup created
[ ] Jenkinsfile updated
[ ] pipeline understood
[ ] agent understood
[ ] environment understood
[ ] parameters understood
[ ] options understood
[ ] stages understood
[ ] stage understood
[ ] steps understood
[ ] script understood
[ ] when understood
[ ] post understood
[ ] Validate stage created
[ ] Build stage created
[ ] Test stage created
[ ] Package stage created
[ ] Archive stage created
[ ] Production conditional stage created
[ ] Pipeline parameters tested
[ ] dev environment tested
[ ] test environment tested
[ ] prod environment tested
[ ] Pipeline failure tested
[ ] Pipeline failure fixed
[ ] Build artifact verified
[ ] Jenkins workspace verified
[ ] Git history checked
[ ] Lab 6 documentation saved
[ ] Lab 6 committed
[ ] Lab 6 pushed to GitHub
```

---

# Lab 6 Cleanup

## Recommended

If you are continuing to Lab 7, do not delete Jenkins.

Keep:

```text
jenkins container
jenkins_home volume
jenkins network
```

Keep:

```text
jenkins-demo-app
```

---

# Restore Lab 5 Jenkinsfile

If you want to return to the previous Jenkinsfile:

```bash
cd /c/project/jenkins-demo-app
```

Restore:

```bash
cp Jenkinsfile.lab5-backup Jenkinsfile
```

Check:

```bash
git status
```

If you want the restored Jenkinsfile in GitHub:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Restore previous Jenkinsfile"
```

Push:

```bash
git push origin main
```

---

# Delete the Jenkins Lab 5 Job

If you no longer need the current job:

```text
Jenkins
→ Dashboard
→ lab-05-pipeline
→ Configure
→ Delete Project
```

You can keep the job instead if you want to reuse it for future labs.

---

# Complete Jenkins Cleanup

Only perform this when you want to completely remove Jenkins.

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
Pipeline Parameters
Artifacts
```

Lab 6:

```text
Advanced Jenkinsfile
Agent
Environment
Parameters
Options
Script
When
Post
Conditional Stages
Pipeline Failure Handling
Pipeline Recovery
```

---

# Next Lab

## Lab 7 — Jenkins CI Pipeline

Now that you understand the Jenkinsfile, we will build a proper CI pipeline.

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
    v
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
Package
    |
    v
Archive
```

In Lab 7, we will make the pipeline more realistic and prepare it for the next major step:

```text
Jenkins → Docker
```

# End of Lab 6
