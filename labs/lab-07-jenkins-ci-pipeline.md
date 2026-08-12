# Jenkins Zero to Hero — Lab 7

## Jenkins CI Pipeline

This lab continues from:

```text
Lab 1 → Jenkins Installation
Lab 2 → Jobs, Workspace and Parameters
Lab 3 → Jenkins + Git + GitHub
Lab 4 → GitHub Webhooks
Lab 5 → Jenkins Pipeline and Jenkinsfile
Lab 6 → Jenkinsfile Deep Dive
```

In this lab, we will build a more realistic **Continuous Integration (CI) pipeline**.

The pipeline will automatically:

```text
GitHub
   |
   | git push
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
   |
   v
CI Success
```

---

# Lab Objective

By the end of this lab, you will understand:

```text
Continuous Integration
CI Pipeline
Automatic GitHub Trigger
Source Checkout
Validation
Build
Testing
Packaging
Artifacts
Build Status
Pipeline Failure
Pipeline Recovery
```

You will also create a CI pipeline that behaves more like a real project.

---

# Important

This lab uses the existing application repository:

```text
jenkins-demo-app
```

and the Jenkins job:

```text
lab-05-pipeline
```

We will improve the Jenkinsfile instead of creating unnecessary repositories.

---

# Prerequisites

Check Jenkins:

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

You may also see:

```text
Jenkinsfile.lab5-backup
```

if you created the backup in Lab 6.

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

If Git reports changes, make sure you know what those changes are before continuing.

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

# Part 4 — Check the Application

Run:

```bash
cat version.txt
```

Example:

```text
4.0
```

Your version may be different.

Run the application:

```bash
chmod +x app.sh
```

Then:

```bash
./app.sh
```

You should see:

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

# Part 5 — Back Up the Existing Jenkinsfile

Create a backup before changing it:

```bash
cp Jenkinsfile Jenkinsfile.lab6-backup
```

Check:

```bash
ls Jenkinsfile*
```

You should see something similar to:

```text
Jenkinsfile
Jenkinsfile.lab5-backup
Jenkinsfile.lab6-backup
```

---

# Part 6 — Create the CI Jenkinsfile

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
    }

    environment {
        APP_NAME = 'jenkins-demo-app'
        BUILD_OUTPUT = 'build'
        CI_VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo '======================================'
                echo 'STAGE: CHECKOUT'
                echo '======================================'

                checkout scm

                sh 'git branch --show-current'
                sh 'git log --oneline -1'
            }
        }

        stage('Validate') {
            steps {
                echo '======================================'
                echo 'STAGE: VALIDATE'
                echo '======================================'

                sh 'test -f Jenkinsfile'
                sh 'test -f app.sh'
                sh 'test -f version.txt'
                sh 'test -f README.md'

                echo 'Required files are present.'

                sh 'test -s app.sh'
                sh 'test -s version.txt'

                echo 'Required files are not empty.'
            }
        }

        stage('Build') {
            steps {
                echo '======================================'
                echo 'STAGE: BUILD'
                echo '======================================'

                echo "Application: ${APP_NAME}"
                echo "CI Build Number: ${CI_VERSION}"

                sh 'chmod +x app.sh'
                sh './app.sh'

                echo 'Application build completed.'
            }
        }

        stage('Test') {
            steps {
                echo '======================================'
                echo 'STAGE: TEST'
                echo '======================================'

                sh 'grep -q "JENKINS DEMO APPLICATION" app.sh'
                sh 'grep -q "Jenkins Demo App" app.sh'
                sh 'grep -q "Application is running successfully." app.sh'

                echo 'Application content tests passed.'

                sh 'test "$(wc -l < version.txt)" -eq 1'

                echo 'Version file test passed.'

                echo 'All tests passed.'
            }
        }

        stage('Package') {
            steps {
                echo '======================================'
                echo 'STAGE: PACKAGE'
                echo '======================================'

                sh 'rm -rf "$BUILD_OUTPUT"'
                sh 'mkdir -p "$BUILD_OUTPUT"'

                sh 'cp app.sh "$BUILD_OUTPUT"/'
                sh 'cp version.txt "$BUILD_OUTPUT"/'
                sh 'cp README.md "$BUILD_OUTPUT"/'

                sh 'echo "Application=${APP_NAME}" > "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "CI_BUILD=${CI_VERSION}" >> "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "JENKINS_BUILD=${BUILD_NUMBER}" >> "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "GIT_COMMIT=$(git rev-parse HEAD)" >> "$BUILD_OUTPUT"/build-info.txt'
                sh 'echo "GIT_BRANCH=$(git branch --show-current)" >> "$BUILD_OUTPUT"/build-info.txt'

                sh 'echo "Package contents:"'
                sh 'find "$BUILD_OUTPUT" -maxdepth 1 -type f -print'
            }
        }

        stage('Archive') {
            steps {
                echo '======================================'
                echo 'STAGE: ARCHIVE'
                echo '======================================'

                archiveArtifacts artifacts: 'build/**', fingerprint: true

                echo 'Artifacts archived successfully.'
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'CI PIPELINE SUCCESS'
            echo '======================================'

            echo "Application: ${APP_NAME}"
            echo "Build Number: ${BUILD_NUMBER}"
            echo "Git Commit: ${GIT_COMMIT}"
        }

        failure {
            echo '======================================'
            echo 'CI PIPELINE FAILED'
            echo '======================================'
        }

        always {
            echo '======================================'
            echo 'CI PIPELINE FINISHED'
            echo '======================================'
        }
    }
}
```

Save the file.

---

# Part 7 — Understand `skipDefaultCheckout(true)`

We added:

```groovy
skipDefaultCheckout(true)
```

This prevents Jenkins from automatically checking out the source code before our stages begin.

We will explicitly perform checkout here:

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

This makes the pipeline easier to understand.

---

# Part 8 — Check the Jenkinsfile

Run:

```bash
cat Jenkinsfile
```

Make sure the pipeline contains:

```text
Checkout
Validate
Build
Test
Package
Archive
```

---

# Part 9 — Validate Required Files Locally

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

# Part 10 — Check the Application Script

Run:

```bash
grep -q "JENKINS DEMO APPLICATION" app.sh && echo "Application header check passed"
```

Expected:

```text
Application header check passed
```

Run:

```bash
grep -q "Application is running successfully." app.sh && echo "Application success check passed"
```

Expected:

```text
Application success check passed
```

---

# Part 11 — Test the Version File

Run:

```bash
test "$(wc -l < version.txt)" -eq 1 && echo "Version file check passed"
```

Expected:

```text
Version file check passed
```

---

# Part 12 — Run the Application

Run:

```bash
chmod +x app.sh
```

Then:

```bash
./app.sh
```

Make sure the application completes successfully.

---

# Part 13 — Commit the CI Pipeline

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
git commit -m "Build Jenkins continuous integration pipeline"
```

---

# Part 14 — Push the CI Pipeline

Run:

```bash
git push origin main
```

---

# Part 15 — Open Jenkins

Open:

```text
http://localhost:8081
```

Open:

```text
lab-05-pipeline
```

The job is configured as:

```text
Pipeline script from SCM
```

so Jenkins will read the updated Jenkinsfile from GitHub.

---

# Part 16 — Run the Pipeline

Click:

```text
Build with Parameters
```

If your current Jenkinsfile does not contain parameters, you may see:

```text
Build Now
```

Use whichever option Jenkins shows.

Start the build.

---

# Part 17 — Check the Pipeline Stages

Open the new build.

You should see:

```text
Checkout
Validate
Build
Test
Package
Archive
```

The stages should execute in that order.

---

# Part 18 — Check Console Output

Click:

```text
Console Output
```

You should see:

```text
STAGE: CHECKOUT
```

Then:

```text
STAGE: VALIDATE
```

Then:

```text
STAGE: BUILD
```

Then:

```text
STAGE: TEST
```

Then:

```text
STAGE: PACKAGE
```

Then:

```text
STAGE: ARCHIVE
```

Finally:

```text
CI PIPELINE SUCCESS
```

---

# Part 19 — Check Git Information

The Checkout stage prints:

```text
git branch --show-current
```

and:

```text
git log --oneline -1
```

In the console you should see the branch and the latest commit.

This proves Jenkins checked out the GitHub source code.

---

# Part 20 — Check the Build Directory

The Package stage creates:

```text
build
```

with:

```text
app.sh
version.txt
README.md
build-info.txt
```

Check from Git Bash:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline/build"
```

Expected:

```text
app.sh
version.txt
README.md
build-info.txt
```

---

# Part 21 — Read Build Information

Run:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-05-pipeline/build/build-info.txt"
```

Expected format:

```text
Application=jenkins-demo-app
CI_BUILD=<build-number>
JENKINS_BUILD=<build-number>
GIT_COMMIT=<commit-hash>
GIT_BRANCH=main
```

The exact values will be different.

---

# Part 22 — Check Build Artifact

Open the Jenkins build.

Find:

```text
Build Artifacts
```

You should see:

```text
build/
```

Open it.

You should see:

```text
app.sh
version.txt
README.md
build-info.txt
```

---

# Part 23 — Understand Continuous Integration

Continuous Integration means developers regularly push code into a shared source repository and an automated system validates the change.

Our current system:

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

---

# Part 24 — Trigger the CI Pipeline with a Git Push

Now we will test the full CI workflow.

Open:

```bash
cd /c/project/jenkins-demo-app
```

Open the README:

```bash
code README.md
```

Add this line:

```text
CI pipeline test completed.
```

Save.

---

# Part 25 — Check Git Status

Run:

```bash
git status
```

You should see:

```text
modified: README.md
```

---

# Part 26 — Commit the Change

Run:

```bash
git add README.md
```

Commit:

```bash
git commit -m "Test continuous integration pipeline"
```

---

# Part 27 — Push to GitHub

Run:

```bash
git push origin main
```

The expected workflow is:

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
CI Pipeline
```

---

# Part 28 — Check Jenkins Automatically

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

A new build should appear if your GitHub webhook from Lab 4 is still configured and your tunnel is running.

If you are testing only the Jenkins pipeline and the webhook is not running, use:

```text
Build Now
```

---

# Part 29 — Verify the New Commit

Open the latest build:

```text
Console Output
```

Look for:

```text
git log --oneline -1
```

You should see:

```text
Test continuous integration pipeline
```

This proves Jenkins built the latest GitHub commit.

---

# Part 30 — Check the Latest Workspace

Run:

```bash
docker exec jenkins sh -c "cd /var/jenkins_home/workspace/lab-05-pipeline && git log --oneline -1"
```

You should see the latest commit.

---

# Part 31 — Add a Build Number to the Package

Our pipeline already creates:

```text
CI_BUILD
```

and:

```text
JENKINS_BUILD
```

For example:

```text
CI_BUILD=10
JENKINS_BUILD=10
```

This is useful when tracking which build produced an artifact.

---

# Part 32 — Understand Build Traceability

Our package now contains:

```text
Application
CI Build
Jenkins Build
Git Commit
Git Branch
```

This gives us a relationship:

```text
Git Commit
    |
    v
Jenkins Build
    |
    v
Build Artifact
```

For example:

```text
Git Commit:
abc1234

Jenkins Build:
#10

Artifact:
build/
```

This becomes very important in real CI/CD systems.

---

# Part 33 — Test a Pipeline Failure

Now we will intentionally break the validation stage.

Open:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

Inside the `Validate` stage, temporarily add:

```groovy
sh 'test -f file-that-does-not-exist.txt'
```

The stage should look like:

```groovy
stage('Validate') {
    steps {
        echo '======================================'
        echo 'STAGE: VALIDATE'
        echo '======================================'

        sh 'test -f Jenkinsfile'
        sh 'test -f app.sh'
        sh 'test -f version.txt'
        sh 'test -f README.md'
        sh 'test -f file-that-does-not-exist.txt'

        echo 'Required files are present.'
    }
}
```

Save.

---

# Part 34 — Commit the Broken Pipeline

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Test CI pipeline failure"
```

Push:

```bash
git push origin main
```

---

# Part 35 — Run the Pipeline

Open Jenkins.

Run:

```text
Build Now
```

or:

```text
Build with Parameters
```

depending on your Jenkins configuration.

The pipeline should fail during:

```text
Validate
```

---

# Part 36 — Inspect the Failure

Open:

```text
Console Output
```

You should see an error caused by:

```text
file-that-does-not-exist.txt
```

The pipeline should not continue normally to:

```text
Build
Test
Package
Archive
```

The pipeline should finish with:

```text
CI PIPELINE FAILED
```

---

# Part 37 — Fix the Pipeline

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

# Part 38 — Commit the Fix

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Fix CI pipeline validation"
```

Push:

```bash
git push origin main
```

---

# Part 39 — Run the Fixed Pipeline

Open Jenkins:

```text
http://localhost:8081
```

Run:

```text
Build Now
```

The pipeline should now succeed.

---

# Part 40 — Verify Successful CI

Open the latest build.

You should see:

```text
Checkout
Validate
Build
Test
Package
Archive
```

And:

```text
CI PIPELINE SUCCESS
```

---

# Part 41 — Check Build History

Open:

```text
lab-05-pipeline
```

You should now have multiple builds.

For example:

```text
#10   SUCCESS
#9    FAILURE
#8    SUCCESS
```

The exact build numbers will depend on your previous labs.

This is useful because Jenkins keeps the history of what happened.

---

# Part 42 — Understand CI Failure

The CI pipeline should stop when an important stage fails.

For example:

```text
Checkout
    |
    v
Validate
    |
    X
Failure
```

Jenkins should not package invalid source code.

That is one of the main purposes of CI.

---

# Part 43 — Understand CI Success

A successful pipeline is:

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
SUCCESS
```

Only code that passes the required checks reaches the artifact stage.

---

# Part 44 — Inspect the Final Artifact

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-05-pipeline/build"
```

Read:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-05-pipeline/build/build-info.txt"
```

Expected format:

```text
Application=jenkins-demo-app
CI_BUILD=<build-number>
JENKINS_BUILD=<build-number>
GIT_COMMIT=<commit-hash>
GIT_BRANCH=main
```

---

# Part 45 — Check Git Status

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

# Part 46 — Check Git History

Run:

```bash
git log --oneline --max-count=10
```

You should see commits similar to:

```text
Fix CI pipeline validation
Test CI pipeline failure
Test continuous integration pipeline
Build Jenkins continuous integration pipeline
```

Your exact history will be different.

---

# Part 47 — Check Application Repository

Run:

```bash
ls -la
```

You should have:

```text
Jenkinsfile
README.md
app.sh
version.txt
```

---

# Part 48 — Understand the CI Pipeline

The final CI process is:

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
    |
    v
CI Success
```

---

# Part 49 — Why This Is CI

We now have:

```text
Source Control
+
Automatic Trigger
+
Automated Validation
+
Automated Build
+
Automated Tests
+
Artifact Creation
```

That is the foundation of a Continuous Integration pipeline.

---

# Part 50 — Save Lab 7 Documentation

Go to your Jenkins documentation repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go to labs:

```bash
cd labs
```

Create the Lab 7 file:

```bash
touch lab-07-jenkins-ci-pipeline.md
```

Open:

```bash
code lab-07-jenkins-ci-pipeline.md
```

Paste this entire lab into the file.

Save.

---

# Part 51 — Check Git Status

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
new file: labs/lab-07-jenkins-ci-pipeline.md
```

---

# Part 52 — Add Lab 7

```bash
git add labs/lab-07-jenkins-ci-pipeline.md
```

Check:

```bash
git status
```

---

# Part 53 — Commit Lab 7

```bash
git commit -m "Add Jenkins Lab 7 CI pipeline"
```

---

# Part 54 — Push Lab 7

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

# Part 55 — Verify Jenkins Repository

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
    └── lab-07-jenkins-ci-pipeline.md
```

---

# Lab 7 Completion Checklist

```text
[ ] Jenkins is running
[ ] Application repository verified
[ ] GitHub remote verified
[ ] Jenkinsfile backed up
[ ] CI Jenkinsfile created
[ ] Checkout stage created
[ ] Validate stage created
[ ] Build stage created
[ ] Test stage created
[ ] Package stage created
[ ] Archive stage created
[ ] CI pipeline executed
[ ] Git information verified
[ ] Build artifacts created
[ ] Build artifacts inspected
[ ] Git push triggered Jenkins
[ ] Latest Git commit verified in Jenkins
[ ] Pipeline failure tested
[ ] Pipeline failure fixed
[ ] Successful CI build verified
[ ] Build history checked
[ ] Build traceability understood
[ ] Lab 7 documentation saved
[ ] Lab 7 committed
[ ] Lab 7 pushed to GitHub
```

---

# Lab 7 Cleanup

## Recommended

Do not remove Jenkins if you are continuing to Lab 8.

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

# Optional Jenkins Job Cleanup

Do not delete:

```text
lab-05-pipeline
```

if you want to continue using the same CI pipeline.

If you want to remove it:

```text
Jenkins
→ Dashboard
→ lab-05-pipeline
→ Configure
→ Delete Project
```

---

# Restore Previous Jenkinsfile

If you need to restore the Lab 6 version:

```bash
cd /c/project/jenkins-demo-app
```

Restore:

```bash
cp Jenkinsfile.lab6-backup Jenkinsfile
```

Check:

```bash
git status
```

If you want to commit the restoration:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Restore Lab 6 Jenkinsfile"
```

Push:

```bash
git push origin main
```

---

# Complete Jenkins Cleanup

Only use these commands when you want to completely remove Jenkins.

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
Automated Checkout
Validation
Build
Testing
Packaging
Artifact Creation
Build Traceability
CI Failure
CI Recovery
```

---

# Next Lab

## Lab 8 — Jenkins + Docker

Now we will connect Jenkins to Docker.

The pipeline will become:

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

We will build a Docker image from Jenkins and learn how Jenkins interacts with Docker.

# End of Lab 7
