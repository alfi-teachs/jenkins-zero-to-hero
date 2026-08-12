# Jenkins Zero to Hero — Lab 5

## Jenkins Pipeline and Jenkinsfile

This lab continues from:

```text
Lab 1 → Jenkins Installation
Lab 2 → Jobs, Workspace and Parameters
Lab 3 → Jenkins + Git + GitHub
Lab 4 → GitHub Webhooks
```

In the previous labs, we used:

```text
Freestyle Project
```

Now we will learn:

```text
Pipeline
```

A Jenkins Pipeline lets us define the CI/CD process as code.

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
Pipeline
    |
    +---- Checkout
    |
    +---- Build
    |
    +---- Test
    |
    +---- Package
    |
    v
Success
```

---

# Lab Objective

By the end of this lab, you will understand:

```text
Jenkins Pipeline
Jenkinsfile
Declarative Pipeline
pipeline
agent
stages
stage
steps
echo
sh
environment
post
Pipeline Console Output
Pipeline from SCM
```

You will also create a real:

```text
GitHub → Jenkins Pipeline
```

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

# Part 1 — Verify the Application Repository

We will continue using:

```text
jenkins-demo-app
```

Go to the application repository:

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

You should have something similar to:

```text
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

# Part 4 — Create the Jenkinsfile

Go to:

```bash
cd /c/project/jenkins-demo-app
```

Create the Jenkinsfile:

```bash
touch Jenkinsfile
```

Check:

```bash
ls
```

You should now see:

```text
Jenkinsfile
README.md
app.sh
version.txt
```

---

# Part 5 — Open the Jenkinsfile

Open:

```bash
code Jenkinsfile
```

Paste:

```groovy
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Starting build...'
                sh 'echo "Building Jenkins Demo App..."'
                sh 'cat version.txt'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'test -f app.sh'
                sh 'test -f version.txt'
                echo 'Tests passed.'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging application...'
                sh 'mkdir -p build'
                sh 'cp app.sh build/'
                sh 'cp version.txt build/'
                sh 'ls -la build'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
```

Save the file.

---

# Part 6 — Understand the Jenkinsfile

The first line:

```groovy
pipeline {
```

means this is a Jenkins Declarative Pipeline.

---

# Part 7 — Understand agent

This line:

```groovy
agent any
```

means Jenkins can run the pipeline on any available Jenkins agent.

For our current setup, Jenkins will use the available built-in node.

---

# Part 8 — Understand stages

We created:

```text
Checkout
Build
Test
Package
```

The pipeline executes them in order:

```text
Checkout
   |
   v
Build
   |
   v
Test
   |
   v
Package
```

---

# Part 9 — Understand steps

Inside a stage we use:

```groovy
steps {
}
```

The steps contain the commands Jenkins should execute.

For example:

```groovy
sh 'echo "Hello Jenkins"'
```

runs a shell command.

---

# Part 10 — Test the Jenkinsfile Locally

We can inspect the file before pushing it.

Run:

```bash
cat Jenkinsfile
```

You should see the pipeline code.

Check that the file exists:

```bash
test -f Jenkinsfile && echo "Jenkinsfile exists"
```

Expected:

```text
Jenkinsfile exists
```

---

# Part 11 — Test the Application

Run:

```bash
sh app.sh
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

# Part 12 — Check Version

Run:

```bash
cat version.txt
```

Example:

```text
4.0
```

---

# Part 13 — Commit the Jenkinsfile

Check:

```bash
git status
```

You should see:

```text
Untracked files:
    Jenkinsfile
```

Add it:

```bash
git add Jenkinsfile
```

Check:

```bash
git status
```

Commit:

```bash
git commit -m "Add Jenkins pipeline"
```

---

# Part 14 — Push Jenkinsfile to GitHub

Run:

```bash
git push origin main
```

---

# Part 15 — Verify Jenkinsfile on GitHub

Open:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app
```

You should now see:

```text
Jenkinsfile
README.md
app.sh
version.txt
```

Click:

```text
Jenkinsfile
```

You should see your pipeline code.

---

# Part 16 — Create Jenkins Pipeline Job

Open Jenkins:

```text
http://localhost:8081
```

Click:

```text
New Item
```

Enter:

```text
lab-05-pipeline
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

# Part 17 — Configure Pipeline from SCM

Scroll down to:

```text
Pipeline
```

For:

```text
Definition
```

select:

```text
Pipeline script from SCM
```

For:

```text
SCM
```

select:

```text
Git
```

For:

```text
Repository URL
```

enter:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app.git
```

For a public repository:

```text
Credentials:
- none -
```

For:

```text
Branch Specifier
```

enter:

```text
*/main
```

For:

```text
Script Path
```

enter:

```text
Jenkinsfile
```

Click:

```text
Save
```

---

# Part 18 — Run the Pipeline

Open:

```text
lab-05-pipeline
```

Click:

```text
Build Now
```

Jenkins should:

```text
Connect to GitHub
      |
      v
Checkout repository
      |
      v
Find Jenkinsfile
      |
      v
Execute Pipeline
```

---

# Part 19 — Open Console Output

Open the new build.

For example:

```text
#1
```

Click:

```text
Console Output
```

You should see something similar to:

```text
Checking out source code...
Starting build...
Building Jenkins Demo App...
Running tests...
Tests passed.
Packaging application...
Pipeline completed successfully!
```

---

# Part 20 — Understand Pipeline Stages

Jenkins should show the stages:

```text
Checkout
Build
Test
Package
```

The pipeline runs them in this order:

```text
Checkout
    |
    v
Build
    |
    v
Test
    |
    v
Package
```

---

# Part 21 — Check the Pipeline Workspace

Open Git Bash.

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline"
```

You should see:

```text
Jenkinsfile
README.md
app.sh
version.txt
build
```

---

# Part 22 — Check the Build Directory

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline/build"
```

Expected:

```text
app.sh
version.txt
```

---

# Part 23 — Read the Version from Jenkins Workspace

Run:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-05-pipeline/version.txt"
```

The output should match the version in GitHub.

---

# Part 24 — Add an Environment Variable to the Pipeline

We will now add a Jenkins environment variable.

Open the Jenkinsfile:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

Replace the current Jenkinsfile with:

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'jenkins-demo-app'
        APP_ENV = 'development'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Application: ${APP_NAME}"
                echo "Environment: ${APP_ENV}"
                echo 'Starting build...'

                sh 'echo "Building Jenkins Demo App..."'
                sh 'cat version.txt'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'

                sh 'test -f app.sh'
                sh 'test -f version.txt'

                echo 'Tests passed.'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging application...'

                sh 'rm -rf build'
                sh 'mkdir -p build'

                sh 'cp app.sh build/'
                sh 'cp version.txt build/'

                sh 'ls -la build'
            }
        }
    }

    post {

        success {
            echo "Pipeline for ${APP_NAME} completed successfully."
        }

        failure {
            echo "Pipeline for ${APP_NAME} failed."
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
```

Save.

---

# Part 25 — Commit the Updated Jenkinsfile

Check:

```bash
git status
```

Add:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add Jenkins pipeline environment variables"
```

Push:

```bash
git push origin main
```

---

# Part 26 — Run the Pipeline Again

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
Build Now
```

Open the new build.

Click:

```text
Console Output
```

You should see:

```text
Application: jenkins-demo-app
Environment: development
```

---

# Part 27 — Understand environment

This section:

```groovy
environment {
    APP_NAME = 'jenkins-demo-app'
    APP_ENV = 'development'
}
```

creates variables available to pipeline stages.

The values can be used as:

```groovy
${APP_NAME}
```

and:

```groovy
${APP_ENV}
```

---

# Part 28 — Add a Deploy Simulation Stage

We are not deploying to Kubernetes yet.

We will create a fake deployment stage so you understand pipeline structure.

Open:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

Add this stage after the `Package` stage:

```groovy
        stage('Deploy') {
            steps {
                echo "Deploying ${APP_NAME}..."
                echo "Target environment: ${APP_ENV}"

                sh 'echo "Deployment simulated successfully."'
            }
        }
```

Your pipeline stages will now be:

```text
Checkout
Build
Test
Package
Deploy
```

Save.

---

# Part 29 — Commit Deploy Stage

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add deployment stage to Jenkins pipeline"
```

Push:

```bash
git push origin main
```

---

# Part 30 — Run the Pipeline

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
Build Now
```

Open the latest build.

Check:

```text
Console Output
```

You should see:

```text
Checkout
Build
Test
Package
Deploy
```

The deploy output should contain:

```text
Deployment simulated successfully.
```

---

# Part 31 — Add Pipeline Parameters

Now we will make the Pipeline accept parameters.

Open:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

Replace the Jenkinsfile with:

```groovy
pipeline {
    agent any

    parameters {
        string(
            name: 'APP_VERSION',
            defaultValue: '1.0',
            description: 'Application version'
        )

        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'test', 'prod'],
            description: 'Deployment environment'
        )
    }

    environment {
        APP_NAME = 'jenkins-demo-app'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Application: ${APP_NAME}"
                echo "Version: ${params.APP_VERSION}"
                echo "Environment: ${params.ENVIRONMENT}"

                sh 'echo "Building application..."'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'

                sh 'test -f app.sh'
                sh 'test -f version.txt'

                echo 'Tests passed.'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging application...'

                sh 'rm -rf build'
                sh 'mkdir -p build'

                sh 'cp app.sh build/'
                sh 'cp version.txt build/'

                sh 'echo "Version=${APP_VERSION}" > build/build-info.txt'
                sh 'echo "Environment=${ENVIRONMENT}" >> build/build-info.txt'

                sh 'cat build/build-info.txt'
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying ${APP_NAME}"
                echo "Version: ${params.APP_VERSION}"
                echo "Environment: ${params.ENVIRONMENT}"

                sh 'echo "Deployment simulated successfully."'
            }
        }
    }

    post {

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            echo "Pipeline execution finished."
        }
    }
}
```

Save.

---

# Part 32 — Commit Pipeline Parameters

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Add Jenkins pipeline parameters"
```

Push:

```bash
git push origin main
```

---

# Part 33 — Run with Parameters

Open Jenkins:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

You should now see:

```text
Build with Parameters
```

Click:

```text
Build with Parameters
```

Enter:

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

# Part 34 — Check Pipeline Output

Open:

```text
Console Output
```

You should see:

```text
Application: jenkins-demo-app
Version: 1.0
Environment: dev
```

You should also see:

```text
Deployment simulated successfully.
```

---

# Part 35 — Test the Test Environment

Click:

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

Expected:

```text
Version: 2.0
Environment: test
```

---

# Part 36 — Test the Production Environment

Click:

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

Expected:

```text
Version: 3.0
Environment: prod
```

---

# Part 37 — Archive Pipeline Artifacts

We created:

```text
build/build-info.txt
```

We will now archive it.

Open Jenkins:

```text
lab-05-pipeline
→ Configure
```

Find:

```text
Post-build Actions
```

Click:

```text
Add post-build action
```

Select:

```text
Archive the artifacts
```

Enter:

```text
build/**
```

Click:

```text
Save
```

---

# Part 38 — Run the Pipeline Again

Click:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
4.0
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

Open the new build.

You should see the artifact:

```text
build/
```

Inside it:

```text
app.sh
version.txt
build-info.txt
```

---

# Part 39 — Inspect the Artifact

Open:

```text
build-info.txt
```

You should see something similar to:

```text
Version=4.0
Environment=dev
```

---

# Part 40 — Understand the Pipeline Structure

Your Jenkinsfile now follows:

```text
pipeline
   |
   +--- parameters
   |
   +--- environment
   |
   +--- stages
          |
          +--- Checkout
          |
          +--- Build
          |
          +--- Test
          |
          +--- Package
          |
          +--- Deploy
   |
   +--- post
```

---

# Part 41 — Understand Declarative Pipeline

Our Jenkinsfile begins with:

```groovy
pipeline {
```

This is a:

```text
Declarative Pipeline
```

It gives us a structured way to describe the CI/CD process.

---

# Part 42 — Important Jenkins Pipeline Keywords

## pipeline

```groovy
pipeline {
}
```

Defines the entire pipeline.

## agent

```groovy
agent any
```

Defines where the pipeline can run.

## stages

```groovy
stages {
}
```

Contains the pipeline stages.

## stage

```groovy
stage('Build') {
}
```

Defines one stage.

## steps

```groovy
steps {
}
```

Contains commands executed by the stage.

## sh

```groovy
sh 'echo "Hello"'
```

Runs a shell command.

## environment

```groovy
environment {
}
```

Defines environment variables.

## parameters

```groovy
parameters {
}
```

Defines values supplied when the build starts.

## post

```groovy
post {
}
```

Defines actions after the pipeline completes.

---

# Part 43 — Test a Pipeline Failure

We will intentionally create a failed build so you can understand Jenkins failure handling.

Open:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

Inside the `Test` stage, temporarily change:

```groovy
sh 'test -f app.sh'
```

to:

```groovy
sh 'test -f file-that-does-not-exist.txt'
```

Save.

---

# Part 44 — Commit the Intentional Failure

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Test Jenkins pipeline failure"
```

Push:

```bash
git push origin main
```

---

# Part 45 — Run the Failed Pipeline

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

Choose:

```text
APP_VERSION:
5.0
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
Test
```

---

# Part 46 — Check the Failed Build

Open:

```text
Console Output
```

You should see an error related to:

```text
file-that-does-not-exist.txt
```

The pipeline should show:

```text
Build failed
```

The `post` section should also execute:

```text
Pipeline failed.
```

---

# Part 47 — Fix the Pipeline

Open:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

Change:

```groovy
sh 'test -f file-that-does-not-exist.txt'
```

back to:

```groovy
sh 'test -f app.sh'
```

Save.

---

# Part 48 — Commit the Fix

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Fix Jenkins pipeline test"
```

Push:

```bash
git push origin main
```

---

# Part 49 — Run Successful Pipeline Again

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
5.1
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

The pipeline should now succeed.

---

# Part 50 — Check Pipeline History

Open:

```text
lab-05-pipeline
```

You should see multiple builds.

For example:

```text
#1
#2
#3
#4
#5
```

You should have at least:

```text
Successful builds
Failed build
Successful build
```

This demonstrates how Jenkins keeps build history.

---

# Part 51 — Check Git Log

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
Add Jenkins pipeline
Add Jenkins pipeline environment variables
Add deployment stage
Add Jenkins pipeline parameters
Test Jenkins pipeline failure
Fix Jenkins pipeline test
```

Your exact commit hashes will be different.

---

# Part 52 — Check Git Status

Run:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# Part 53 — Inspect Pipeline Workspace

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline"
```

Check the build directory:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline/build"
```

Read:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-05-pipeline/build/build-info.txt"
```

---

# Part 54 — Important Difference: Freestyle vs Pipeline

Freestyle:

```text
Jenkins UI
    |
    v
Build Step
    |
    v
Shell Commands
```

Pipeline:

```text
GitHub
    |
    v
Jenkins
    |
    v
Jenkinsfile
    |
    +--- Checkout
    |
    +--- Build
    |
    +--- Test
    |
    +--- Package
    |
    +--- Deploy
```

Pipeline configuration is stored as code.

That is a major advantage.

---

# Part 55 — Why Jenkinsfile Matters

Instead of manually remembering:

```text
What should Jenkins build?
What tests should it run?
What should happen after the build?
```

we store the process in:

```text
Jenkinsfile
```

The Jenkinsfile can be version-controlled with the application.

That means:

```text
Application Code
+
Jenkins Pipeline Code
```

are both stored in Git.

---

# Part 56 — The Complete CI Pipeline

You now have:

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
    +---- Build
    |
    +---- Test
    |
    +---- Package
    |
    +---- Deploy
    |
    v
Artifact
```

---

# Part 57 — Save Lab 5 Documentation

Go to your Jenkins documentation repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go into labs:

```bash
cd labs
```

Create the Lab 5 file if necessary:

```bash
touch lab-05-jenkins-pipeline.md
```

Open:

```bash
code lab-05-jenkins-pipeline.md
```

Paste this complete lab into the file.

Save.

---

# Part 58 — Check Git Status

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
new file: labs/lab-05-jenkins-pipeline.md
```

or:

```text
modified: labs/lab-05-jenkins-pipeline.md
```

---

# Part 59 — Add Lab 5

```bash
git add labs/lab-05-jenkins-pipeline.md
```

Check:

```bash
git status
```

---

# Part 60 — Commit Lab 5

```bash
git commit -m "Add Jenkins Lab 5 pipeline and Jenkinsfile"
```

---

# Part 61 — Push Lab 5 to GitHub

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

# Part 62 — Verify Repository Structure

Your Jenkins documentation repository should now be:

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
    └── lab-05-jenkins-pipeline.md
```

---

# Lab 5 Completion Checklist

```text
[ ] Jenkins is running
[ ] Jenkinsfile created
[ ] Jenkinsfile pushed to GitHub
[ ] Pipeline Jenkins job created
[ ] Pipeline configured from SCM
[ ] GitHub repository configured
[ ] Jenkinsfile path configured
[ ] Pipeline successfully executed
[ ] Checkout stage completed
[ ] Build stage completed
[ ] Test stage completed
[ ] Package stage completed
[ ] Deploy stage completed
[ ] Environment variables used
[ ] Pipeline parameters added
[ ] Parameterized build tested
[ ] Development environment tested
[ ] Test environment tested
[ ] Production environment tested
[ ] Build artifacts archived
[ ] Pipeline failure tested
[ ] Pipeline failure fixed
[ ] Successful build verified
[ ] Pipeline workspace inspected
[ ] Lab 5 documentation saved
[ ] Lab 5 committed
[ ] Lab 5 pushed to GitHub
```

---

# Lab 5 Cleanup

## Recommended

If you are continuing to Lab 6, do not delete the Jenkins installation.

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

# Delete Only the Lab 5 Jenkins Job

From Jenkins:

```text
Dashboard
→ lab-05-pipeline
→ Configure
→ Delete Project
```

Confirm if you no longer need the job.

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
Cloudflare Quick Tunnel
Git Push → Jenkins
```

Lab 5:

```text
Jenkins Pipeline
Jenkinsfile
Declarative Pipeline
Stages
Steps
Agents
Environment Variables
Parameters
Post Actions
Artifacts
Pipeline Failure
Pipeline Recovery
```

---

# Next Lab

## Lab 6 — Jenkinsfile Deep Dive

In Lab 6, we will go deeper into the Jenkinsfile.

We will learn:

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
```

Then we will build a more realistic CI pipeline:

```text
GitHub
    |
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
    +---- Package
    |
    +---- Archive
    |
    v
Success
```

# End of Lab 5
