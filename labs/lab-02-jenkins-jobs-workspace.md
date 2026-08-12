# Jenkins Zero to Hero — Lab 2

## Jenkins Jobs, Workspace, Environment Variables and Build Parameters

This lab continues from **Lab 1 — Install Jenkins**.

Repository:

```text
jenkins-zero-to-hero
```

Lab file:

```text
labs/lab-02-jenkins-jobs-workspace.md
```

---

# Lab Objective

By the end of this lab, you will understand:

- Jenkins jobs
- Jenkins builds
- Build history
- Build numbers
- Jenkins workspace
- Jenkins environment variables
- Custom variables
- Build parameters
- Parameterized builds
- Build artifacts
- Jenkins workspace inspection using Docker

---

# Prerequisites

Jenkins should already be installed from Lab 1.

Check Jenkins:

```bash
docker ps
```

Expected output:

```text
CONTAINER ID   IMAGE                 NAMES
xxxxxxxxxxxx   jenkins/jenkins:lts  jenkins
```

If Jenkins is stopped:

```bash
docker start jenkins
```

Check again:

```bash
docker ps
```

Open Jenkins:

```text
http://localhost:8081
```

---

# Part 1 — Create the Lab 2 Jenkins Job

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
lab-02-job
```

Select:

```text
Freestyle project
```

Click:

```text
OK
```

---

# Part 2 — Create the First Build

Scroll to:

```text
Build Steps
```

Click:

```text
Add build step
```

Select:

```text
Execute shell
```

Paste:

```bash
echo "======================================"
echo "JENKINS LAB 2"
echo "======================================"

echo "Hello from Jenkins!"

echo "Job Name:"
echo "$JOB_NAME"

echo "Build Number:"
echo "$BUILD_NUMBER"

echo "Build ID:"
echo "$BUILD_ID"

echo "Workspace:"
echo "$WORKSPACE"

echo "Node Name:"
echo "$NODE_NAME"

echo "Jenkins URL:"
echo "$JENKINS_URL"

echo "Build URL:"
echo "$BUILD_URL"

echo "======================================"
```

Click:

```text
Save
```

---

# Part 3 — Run the Jenkins Job

Click:

```text
Build Now
```

You should see:

```text
Build #1
```

Click:

```text
#1
```

Then:

```text
Console Output
```

Expected output will look similar to:

```text
======================================
JENKINS LAB 2
======================================

Hello from Jenkins!

Job Name:
lab-02-job

Build Number:
1

Build ID:
1

Workspace:
/var/jenkins_home/workspace/lab-02-job

Node Name:
built-in

Jenkins URL:
http://localhost:8081/

Build URL:
http://localhost:8081/job/lab-02-job/1/

======================================
```

The exact values can be different.

---

# Part 4 — Create Multiple Builds

Go back to:

```text
lab-02-job
```

Click:

```text
Build Now
```

Then:

```text
Build Now
```

You should now have:

```text
#1
#2
#3
```

Jenkins automatically increases:

```text
BUILD_NUMBER
```

Example:

```text
Build #1 → BUILD_NUMBER=1
Build #2 → BUILD_NUMBER=2
Build #3 → BUILD_NUMBER=3
```

---

# Part 5 — Check Build History

On the Jenkins job page, find:

```text
Build History
```

You should see:

```text
#3
#2
#1
```

Click:

```text
#3
```

Then:

```text
Console Output
```

This shows what Jenkins executed during that build.

---

# Part 6 — Understand Job and Build

A Jenkins **job** is the configured task.

A Jenkins **build** is one execution of that job.

Example:

```text
Job:
lab-02-job

Builds:
#1
#2
#3
#4
```

The job remains available.

Every execution creates a new build.

---

# Part 7 — Understand the Jenkins Workspace

The workspace for this job will normally be:

```text
/var/jenkins_home/workspace/lab-02-job
```

Jenkins uses the workspace to:

```text
Download source code
Create files
Build applications
Run tests
Generate output
```

Later, GitHub source code will be checked out into this workspace.

---

# Part 8 — Create Files in the Workspace

Open:

```text
lab-02-job
→ Configure
```

Go to:

```text
Build Steps
```

Select:

```text
Execute shell
```

Replace the existing script with:

```bash
echo "======================================"
echo "CREATING WORKSPACE FILES"
echo "======================================"

echo "This file was created by Jenkins." > hello.txt

echo "Jenkins Zero to Hero Lab 2" > lab-info.txt

mkdir -p output

echo "Build completed successfully." > output/result.txt

echo "Files created:"
find . -type f -print

echo "======================================"
```

Click:

```text
Save
```

Click:

```text
Build Now
```

---

# Part 9 — Check the Workspace in Jenkins

Open:

```text
lab-02-job
```

Click:

```text
Workspace
```

You should see:

```text
hello.txt
lab-info.txt
output
```

Open:

```text
output
```

You should see:

```text
result.txt
```

---

# Part 10 — Inspect the Workspace Using Docker

Open Git Bash.

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-02-job"
```

Expected output will contain:

```text
hello.txt
lab-info.txt
output
```

Check the output directory:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-02-job/output"
```

Expected:

```text
result.txt
```

Read the file:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-02-job/output/result.txt"
```

Expected:

```text
Build completed successfully.
```

---

# Part 11 — Jenkins Environment Variables

Jenkins provides environment variables automatically.

Important examples:

```text
JOB_NAME
BUILD_NUMBER
BUILD_ID
BUILD_TAG
WORKSPACE
NODE_NAME
JENKINS_URL
BUILD_URL
```

These variables tell the build information about the current Jenkins execution.

---

# Part 12 — Display Jenkins Environment Variables

Go to:

```text
lab-02-job
→ Configure
→ Build Steps
→ Execute shell
```

Paste:

```bash
echo "======================================"
echo "JENKINS ENVIRONMENT VARIABLES"
echo "======================================"

echo "JOB_NAME=$JOB_NAME"
echo "BUILD_NUMBER=$BUILD_NUMBER"
echo "BUILD_ID=$BUILD_ID"
echo "BUILD_TAG=$BUILD_TAG"
echo "WORKSPACE=$WORKSPACE"
echo "NODE_NAME=$NODE_NAME"
echo "JENKINS_URL=$JENKINS_URL"
echo "BUILD_URL=$BUILD_URL"

echo "======================================"
```

Click:

```text
Save
```

Run:

```text
Build Now
```

Open:

```text
Console Output
```

---

# Part 13 — Display All Environment Variables

Replace the shell script with:

```bash
echo "All environment variables:"
env | sort
```

Click:

```text
Save
```

Run:

```text
Build Now
```

Open:

```text
Console Output
```

To show only common Jenkins variables:

```bash
env | grep -E 'JENKINS|BUILD|JOB|WORKSPACE|NODE'
```

---

# Part 14 — Create Custom Variables

We can define our own environment variables inside the shell script.

Use:

```bash
export APP_NAME="jenkins-demo"
export APP_VERSION="1.0"

echo "Application Name: $APP_NAME"
echo "Application Version: $APP_VERSION"
echo "Application: $APP_NAME:$APP_VERSION"
```

Run the build.

Expected:

```text
Application Name: jenkins-demo
Application Version: 1.0
Application: jenkins-demo:1.0
```

---

# Part 15 — Create a Parameterized Jenkins Job

Now we will allow the person starting the build to provide values.

Open:

```text
lab-02-job
→ Configure
```

Enable:

```text
This project is parameterized
```

---

# Part 16 — Add APP_NAME Parameter

Click:

```text
Add Parameter
```

Select:

```text
String Parameter
```

Enter:

```text
Name:
APP_NAME
```

Enter:

```text
Default Value:
my-app
```

Enter:

```text
Description:
Application name
```

---

# Part 17 — Add ENVIRONMENT Parameter

Click:

```text
Add Parameter
```

Select:

```text
Choice Parameter
```

Enter:

```text
Name:
ENVIRONMENT
```

Enter these choices:

```text
dev
test
prod
```

Description:

```text
Deployment environment
```

---

# Part 18 — Add APP_VERSION Parameter

Click:

```text
Add Parameter
```

Select:

```text
String Parameter
```

Enter:

```text
Name:
APP_VERSION
```

Enter:

```text
Default Value:
1.0
```

Description:

```text
Application version
```

Click:

```text
Save
```

---

# Part 19 — Use the Parameters

Go to:

```text
Build Steps
```

Select:

```text
Execute shell
```

Paste:

```bash
echo "======================================"
echo "PARAMETERIZED JENKINS BUILD"
echo "======================================"

echo "Application Name:"
echo "$APP_NAME"

echo "Application Version:"
echo "$APP_VERSION"

echo "Environment:"
echo "$ENVIRONMENT"

echo "Build Number:"
echo "$BUILD_NUMBER"

echo "Workspace:"
echo "$WORKSPACE"

echo "======================================"

mkdir -p output

echo "Application=$APP_NAME" > output/build-info.txt
echo "Version=$APP_VERSION" >> output/build-info.txt
echo "Environment=$ENVIRONMENT" >> output/build-info.txt
echo "Build=$BUILD_NUMBER" >> output/build-info.txt

echo "Build information:"
cat output/build-info.txt

echo "======================================"
echo "BUILD COMPLETED"
echo "======================================"
```

Click:

```text
Save
```

---

# Part 20 — Run Build with Parameters

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
APP_NAME:
my-web-app
```

Choose:

```text
ENVIRONMENT:
dev
```

Enter:

```text
APP_VERSION:
1.0
```

Click:

```text
Build
```

---

# Part 21 — Check the Parameterized Build

Open the new build.

Click:

```text
Console Output
```

Expected:

```text
======================================
PARAMETERIZED JENKINS BUILD
======================================

Application Name:
my-web-app

Application Version:
1.0

Environment:
dev
```

And:

```text
Build information:
Application=my-web-app
Version=1.0
Environment=dev
Build=<build-number>
```

---

# Part 22 — Test the Test Environment

Click:

```text
Build with Parameters
```

Use:

```text
APP_NAME:
payment-service
```

Choose:

```text
ENVIRONMENT:
test
```

Use:

```text
APP_VERSION:
2.5
```

Click:

```text
Build
```

Expected:

```text
Application=payment-service
Version=2.5
Environment=test
```

---

# Part 23 — Test the Production Environment

Click:

```text
Build with Parameters
```

Use:

```text
APP_NAME:
payment-service
```

Choose:

```text
ENVIRONMENT:
prod
```

Use:

```text
APP_VERSION:
3.0
```

Click:

```text
Build
```

Expected:

```text
Application=payment-service
Version=3.0
Environment=prod
```

---

# Part 24 — Archive the Build Artifact

We created:

```text
output/build-info.txt
```

Now save it as a Jenkins artifact.

Open:

```text
lab-02-job
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

For:

```text
Files to archive
```

enter:

```text
output/build-info.txt
```

Click:

```text
Save
```

---

# Part 25 — Test the Artifact

Click:

```text
Build with Parameters
```

Use:

```text
APP_NAME:
frontend
```

Choose:

```text
ENVIRONMENT:
dev
```

Use:

```text
APP_VERSION:
1.0
```

Click:

```text
Build
```

Open the new build.

You should see:

```text
build-info.txt
```

Click it.

Expected content:

```text
Application=frontend
Version=1.0
Environment=dev
Build=<build-number>
```

---

# Part 26 — Inspect the Jenkins Container

Open Git Bash.

Run:

```bash
docker exec -it jenkins sh
```

Inside the container:

```bash
pwd
```

Go to Jenkins home:

```bash
cd /var/jenkins_home
```

List files:

```bash
ls -la
```

Check workspace:

```bash
ls -la workspace
```

Exit:

```bash
exit
```

---

# Part 27 — Inspect the Lab 2 Workspace Again

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-02-job"
```

Check:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-02-job/output"
```

Read:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-02-job/output/build-info.txt"
```

---

# Part 28 — Understand Job vs Build

Remember:

```text
Job = configured Jenkins task
Build = one execution of that task
```

Example:

```text
lab-02-job
    |
    ├── Build #1
    ├── Build #2
    ├── Build #3
    ├── Build #4
    └── Build #5
```

---

# Part 29 — Complete Lab 2 Flow

```text
Jenkins Job
     |
     v
Build with Parameters
     |
     +------------------+
     |                  |
     v                  v
APP_NAME          APP_VERSION
     |                  |
     +--------+---------+
              |
              v
        ENVIRONMENT
              |
              v
        Jenkins Build
              |
              v
          Workspace
              |
              v
       build-info.txt
              |
              v
       Archived Artifact
```

---

# Part 30 — Save Lab 2 to GitHub

Go to the Jenkins repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Check:

```bash
ls
```

Go to the labs folder:

```bash
cd labs
```

Check:

```bash
ls
```

You should see:

```text
lab-01-install-jenkins.md
lab-02-jenkins-jobs-workspace.md
```

Open Lab 2:

```bash
code lab-02-jenkins-jobs-workspace.md
```

Paste this entire lab into the file.

Save the file.

---

# Part 31 — Check Git Status

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
modified: labs/lab-02-jenkins-jobs-workspace.md
```

or:

```text
new file: labs/lab-02-jenkins-jobs-workspace.md
```

---

# Part 32 — Add Lab 2

```bash
git add labs/lab-02-jenkins-jobs-workspace.md
```

Check:

```bash
git status
```

---

# Part 33 — Commit Lab 2

```bash
git commit -m "Add Jenkins Lab 2 jobs workspace and build parameters"
```

---

# Part 34 — Push to GitHub

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

# Part 35 — Verify GitHub

Your repository should now look like:

```text
jenkins-zero-to-hero/
│
├── README.md
│
└── labs/
    │
    ├── lab-01-install-jenkins.md
    │
    └── lab-02-jenkins-jobs-workspace.md
```

---

# Lab 2 Completion Checklist

```text
[ ] Jenkins is running
[ ] lab-02-job created
[ ] First build completed
[ ] Build history checked
[ ] Build numbers understood
[ ] Jenkins workspace inspected
[ ] Workspace files created
[ ] Jenkins environment variables checked
[ ] Custom variables tested
[ ] Parameterized build enabled
[ ] APP_NAME parameter created
[ ] ENVIRONMENT parameter created
[ ] APP_VERSION parameter created
[ ] Multiple parameterized builds completed
[ ] Build artifact created
[ ] Artifact archived
[ ] Jenkins workspace inspected using Docker
[ ] Lab 2 Markdown saved
[ ] Lab committed to Git
[ ] Lab pushed to GitHub
```

---

# Lab 2 Cleanup

## Recommended Cleanup

If you are continuing to Lab 3, do not delete Jenkins.

Keep:

```text
jenkins container
jenkins_home volume
jenkins network
```

We will reuse them.

---

# Delete Only the Lab 2 Jenkins Job

From Jenkins:

```text
Dashboard
→ lab-02-job
→ Configure
→ Delete Project
```

Confirm deletion.

This removes only the job.

Jenkins itself remains installed.

---

# Complete Jenkins Cleanup

Only use these commands when you want to completely remove Jenkins.

Stop Jenkins:

```bash
docker stop jenkins
```

Remove Jenkins container:

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

Verify containers:

```bash
docker ps -a
```

Verify volumes:

```bash
docker volume ls
```

Verify networks:

```bash
docker network ls
```

---

# What You Learned

Lab 1:

```text
Jenkins
Docker
Jenkins Dashboard
Jenkins Administrator
First Jenkins Job
First Jenkins Build
```

Lab 2:

```text
Jenkins Jobs
Builds
Build History
Build Numbers
Jenkins Workspace
Environment Variables
Custom Variables
Build Parameters
Parameterized Builds
Build Artifacts
Docker Workspace Inspection
```

---

# Next Lab

## Lab 3 — Jenkins + Git + GitHub

The next workflow will be:

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
Git Checkout
    |
    v
Jenkins Workspace
    |
    v
Build
    |
    v
Console Output
```

In Lab 3, Jenkins will pull source code directly from GitHub instead of us manually creating files inside Jenkins.

# End of Lab 2
