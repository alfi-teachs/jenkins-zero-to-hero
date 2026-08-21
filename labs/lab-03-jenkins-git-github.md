# Jenkins Zero to Hero — Lab 3

## Jenkins + Git + GitHub

This lab connects Jenkins to GitHub.

In Lab 2, Jenkins ran shell commands directly.

In Lab 3, Jenkins will:

```text
GitHub
   |
   | Pull source code
   v
Jenkins
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

Repository:

```text
jenkins-zero-to-hero
```

Lab file:

```text
labs/lab-03-jenkins-git-github.md
```

---

# Lab Objective

By the end of this lab, you will know how to:

- Create a GitHub repository for application code
- Add application files to GitHub
- Connect Jenkins to GitHub
- Configure a Jenkins Git job
- Checkout source code from GitHub
- Understand Jenkins workspace after Git checkout
- Build a project from GitHub source code
- Make a Git change
- Push the change to GitHub
- Run Jenkins again
- Verify that Jenkins received the new code

---

# Important Repository Structure

We will use two repositories:

```text
1. Jenkins training repository

jenkins-zero-to-hero
```

This contains all your Jenkins lab documentation.

```text
2. Application source-code repository

jenkins-demo-app
```

This contains the application code that Jenkins will pull and build.

So the final structure is:

```text
GitHub
│
├── jenkins-zero-to-hero
│   └── labs
│       ├── lab-01-install-jenkins.md
│       ├── lab-02-jenkins-jobs-workspace.md
│       └── lab-03-jenkins-git-github.md
│
└── jenkins-demo-app
    ├── README.md
    ├── app.sh
    └── version.txt
```

---

# Prerequisites

Jenkins must already be running.

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

Open Jenkins:

```text
http://localhost:8081
```

---

# Part 1 — Go to Your Jenkins Repository

Open Git Bash.

Go to your Jenkins repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Check:

```bash
pwd
```

Check files:

```bash
ls
```

Go into labs:

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
lab-03-jenkins-git-github.md
```

---

# Part 2 — Create the Lab 3 Markdown File

If the file does not already exist:

```bash
touch lab-03-jenkins-git-github.md
```

Open it:

```bash
code lab-03-jenkins-git-github.md
```

Save this lab into that file.

---

# Part 3 — Create the Application Repository on GitHub

Go to GitHub in your browser.

Create a new repository:

```text
jenkins-demo-app
```

Recommended:

```text
Repository name:
jenkins-demo-app
```

For this lab, make the repository:

```text
Public
```

Do not add:

```text
README
.gitignore
License
```

We will create the files locally.

---

# Part 4 — Clone the Application Repository

Go to your project directory:

```bash
cd /c/project
```

Clone the repository:

```bash
git clone https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app.git
```

Enter the repository:

```bash
cd jenkins-demo-app
```

Check:

```bash
pwd
```

Check files:

```bash
ls
```

The repository should currently be empty.

---

# Part 5 — Create the Application Files

Create the README:

```bash
touch README.md
```

Create the application script:

```bash
touch app.sh
```

Create the version file:

```bash
touch version.txt
```

Check:

```bash
ls
```

Expected:

```text
README.md
app.sh
version.txt
```

---

# Part 6 — Create the Application Script

Open:

```bash
code app.sh
```

Paste:

```bash
#!/bin/sh

echo "======================================"
echo "JENKINS DEMO APPLICATION"
echo "======================================"

echo "Application: Jenkins Demo App"
echo "Environment: Development"

echo "Application is running successfully."

echo "======================================"
```

Save the file.

---

# Part 7 — Add the Application Version

Open:

```bash
code version.txt
```

Paste:

```text
1.0
```

Save.

---

# Part 8 — Create the Application README

Open:

```bash
code README.md
```

Paste:

```markdown
# Jenkins Demo App

This application is used for Jenkins Zero to Hero Lab 3.

The goal is to demonstrate:

- Git
- GitHub
- Jenkins
- Git checkout
- Jenkins workspace
- Jenkins builds
```

Save.

---

# Part 9 — Test the Application Locally

Run:

```bash
sh app.sh
```

Expected output:

```text
======================================
JENKINS DEMO APPLICATION
======================================

Application: Jenkins Demo App
Environment: Development

Application is running successfully.

======================================
```

Check the version:

```bash
cat version.txt
```

Expected:

```text
1.0
```

---

# Part 10 — Git Status

Check:

```bash
git status
```

You should see:

```text
README.md
app.sh
version.txt
```

---

# Part 11 — Add the Files to Git

```bash
git add .
```

Check:

```bash
git status
```

The files should now appear under:

```text
Changes to be committed
```

---

# Part 12 — Commit the Application

```bash
git commit -m "Add Jenkins demo application"
```

---

# Part 13 — Push the Application to GitHub

```bash
git push origin main
```

If your branch is not `main`, check:

```bash
git branch
```

If needed, rename the branch:

```bash
git branch -M main
```

Then:

```bash
git push -u origin main
```

---

# Part 14 — Verify GitHub

Open your GitHub repository:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app
```

You should see:

```text
README.md
app.sh
version.txt
```

---

# Part 15 — Create a Jenkins Git Job

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
lab-03-git-checkout
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

# Part 16 — Configure GitHub Repository

Find:

```text
Source Code Management
```

Select:

```text
Git
```

In:

```text
Repository URL
```

enter:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app.git
```

For:

```text
Credentials
```

leave:

```text
- none -
```

when the repository is public.

For:

```text
Branch Specifier
```

use:

```text
*/main
```

---

# Part 17 — Add a Build Step

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
echo "JENKINS GIT CHECKOUT LAB"
echo "======================================"

echo "Job Name:"
echo "$JOB_NAME"

echo "Build Number:"
echo "$BUILD_NUMBER"

echo "Workspace:"
echo "$WORKSPACE"

echo "======================================"
echo "Workspace Files"
echo "======================================"

pwd

ls -la

echo "======================================"
echo "Application README"
echo "======================================"

cat README.md

echo "======================================"
echo "Application Version"
echo "======================================"

cat version.txt

echo "======================================"
echo "Running Application"
echo "======================================"

chmod +x app.sh

./app.sh

echo "======================================"
echo "BUILD COMPLETED"
echo "======================================"
```

Click:

```text
Save
```

---

# Part 18 — Run the Jenkins Git Job

Click:

```text
Build Now
```

Jenkins will:

```text
Connect to GitHub
      |
      v
Clone the repository
      |
      v
Checkout main
      |
      v
Put files in workspace
      |
      v
Run shell commands
```

---

# Part 19 — Check Console Output

Click:

```text
#1
```

Then:

```text
Console Output
```

You should see Git checkout activity similar to:

```text
Started by user
Building in workspace
Cloning the remote Git repository
Fetching upstream changes
Checking out Revision
```

Then your script output:

```text
======================================
JENKINS GIT CHECKOUT LAB
======================================

Job Name:
lab-03-git-checkout

Build Number:
1

Workspace:
/var/jenkins_home/workspace/lab-03-git-checkout
```

Then:

```text
Workspace Files
```

You should see:

```text
README.md
app.sh
version.txt
```

---

# Part 20 — Verify the Git Checkout

The important part is that Jenkins now has the GitHub files in its workspace.

The workspace should contain:

```text
README.md
app.sh
version.txt
.git
```

Jenkins checked these files out automatically from GitHub.

---

# Part 21 — Check the Jenkins Workspace from Docker

Open Git Bash.

Run:

```bash
docker exec -it jenkins sh 
```
```bash
ls -la 
```
```bash
cd /var/jenkins_home/workspace/lab-03-git-checkout
```
You should see:

```text
Jenkinsfile
README.md
app.sh
version.txt
```

---

# Part 22 — Read the Checked-Out File from Docker

Run:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-03-git-checkout/version.txt"
```

Expected:

```text
1.0
```

Read the application script:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-03-git-checkout/app.sh"
```

---

# Part 23 — Change the Application

Now we will change the application source code.

Go to the application repository:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code version.txt
```

Change:

```text
1.0
```

to:

```text
2.0
```

Save.

---

# Part 24 — Change the Application Script

Open:

```bash
code app.sh
```

Change:

```bash
echo "Environment: Development"
```

to:

```bash
echo "Environment: Jenkins CI"
```

Add:

```bash
echo "Version: 2.0"
```

So the file should contain:

```bash
#!/bin/sh

echo "======================================"
echo "JENKINS DEMO APPLICATION"
echo "======================================"

echo "Application: Jenkins Demo App"
echo "Environment: Jenkins CI"
echo "Version: 2.0"

echo "Application is running successfully."

echo "======================================"
```

Save.

---

# Part 25 — Test the New Version Locally

Run:

```bash
sh app.sh
```

Expected:

```text
======================================
JENKINS DEMO APPLICATION
======================================

Application: Jenkins Demo App
Environment: Jenkins CI
Version: 2.0

Application is running successfully.

======================================
```

Check:

```bash
cat version.txt
```

Expected:

```text
2.0
```

---

# Part 26 — Git Status After the Change

Run:

```bash
git status
```

You should see:

```text
modified: app.sh
modified: version.txt
```

---

# Part 27 — Commit the Change

```bash
git add app.sh version.txt
```

Commit:

```bash
git commit -m "Update application to version 2.0"
```

---

# Part 28 — Push the Change to GitHub

```bash
git push origin main
```

---

# Part 29 — Verify the GitHub Change

Refresh the GitHub repository.

You should now see:

```text
version.txt → 2.0
```

and:

```text
app.sh
```

with the new Jenkins CI environment and version output.

---

# Part 30 — Run Jenkins Again

Go back to Jenkins:

```text
http://localhost:8081
```

Open:

```text
lab-03-git-checkout
```

Click:

```text
Build Now
```

This creates:

```text
#2
```

---

# Part 31 — Check Build #2

Click:

```text
#2
```

Then:

```text
Console Output
```

Jenkins should pull the latest GitHub code.

You should now see:

```text
Application Version
2.0
```

And:

```text
Environment: Jenkins CI
Version: 2.0
```

This proves that Jenkins is retrieving the latest version from GitHub.

---

# Part 32 — Compare Build #1 and Build #2

Build #1:

```text
Version:
1.0

Environment:
Development
```

Build #2:

```text
Version:
2.0

Environment:
Jenkins CI
```

The source code changed in GitHub between the two Jenkins builds.

Jenkins pulled the new version during Build #2.

---

# Part 33 — Understand the Jenkins Git Workflow

The complete workflow is now:

```text
Developer
    |
    | git add
    | git commit
    | git push
    v
GitHub
    |
    | Git clone / fetch
    v
Jenkins
    |
    v
Jenkins Workspace
    |
    v
Build Step
    |
    v
Application
```

---

# Part 34 — Why Git Integration Matters

Without Git integration:

```text
Jenkins
   |
   v
Manual files
```

With Git integration:

```text
Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   v
Build
```

This is the foundation of CI/CD.

---

# Part 35 — Check Git Branch

From the application repository:

```bash
cd /c/project/jenkins-demo-app
```

Check:

```bash
git branch
```

Expected:

```text
* main
```

---

# Part 36 — Check Git Remote

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

# Part 37 — Check Git Log

Run:

```bash
git log --oneline --max-count=5
```

You should see commits similar to:

```text
xxxxxxxx Update application to version 2.0
xxxxxxxx Add Jenkins demo application
```

---

# Part 38 — Jenkins Git Configuration Summary

Your Jenkins job should have:

```text
Job Name:
lab-03-git-checkout
```

Source Code Management:

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

Build Step:

```text
Execute shell
```

---

# Part 39 — If Your GitHub Repository Is Private

If `jenkins-demo-app` is private, Jenkins will need GitHub credentials.

Do not put your GitHub password directly into the repository URL.

Instead:

```text
Jenkins
→ Manage Jenkins
→ Credentials
```

Create a GitHub credential using a GitHub Personal Access Token.

Then return to:

```text
lab-03-git-checkout
→ Configure
→ Source Code Management
→ Git
```

Select the credential you created.

The repository URL remains:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app.git
```

---

# Part 40 — Inspect Jenkins Workspace

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-03-git-checkout"
```

Check the version:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-03-git-checkout/version.txt"
```

Expected:

```text
2.0
```

Run the application directly from the workspace:

```bash
docker exec jenkins sh -c "cd /var/jenkins_home/workspace/lab-03-git-checkout && chmod +x app.sh && ./app.sh"
```

---

# Part 41 — Save Lab 3 Documentation

Go to the Jenkins repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go to labs:

```bash
cd labs
```

Open:

```bash
code lab-03-jenkins-git-github.md
```

Paste this complete lab into the file.

Save it.

---

# Part 42 — Check Git Status

Go back to repository root:

```bash
cd /c/project/jenkins-zero-to-hero
```

Run:

```bash
git status
```

You should see:

```text
modified: labs/lab-03-jenkins-git-github.md
```

or:

```text
new file: labs/lab-03-jenkins-git-github.md
```

---

# Part 43 — Add Lab 3

```bash
git add labs/lab-03-jenkins-git-github.md
```

Check:

```bash
git status
```

---

# Part 44 — Commit Lab 3

```bash
git commit -m "Add Jenkins Lab 3 Git and GitHub integration"
```

---

# Part 45 — Push Lab 3 to GitHub

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

# Part 46 — Verify Jenkins Repository

Your Jenkins documentation repository should now look like:

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
    └── lab-03-jenkins-git-github.md
```

---

# Lab 3 Completion Checklist

```text
[ ] Jenkins is running
[ ] jenkins-demo-app GitHub repository created
[ ] Local jenkins-demo-app repository cloned
[ ] README.md created
[ ] app.sh created
[ ] version.txt created
[ ] Application tested locally
[ ] Application pushed to GitHub
[ ] Jenkins Git job created
[ ] GitHub repository configured in Jenkins
[ ] main branch configured
[ ] Jenkins successfully checked out GitHub code
[ ] Jenkins workspace inspected
[ ] Jenkins build completed
[ ] Application version changed from 1.0 to 2.0
[ ] Change committed to Git
[ ] Change pushed to GitHub
[ ] Jenkins Build #2 pulled the new code
[ ] Jenkins workspace verified
[ ] Lab 3 documentation added to Git
[ ] Lab 3 committed
[ ] Lab 3 pushed to GitHub
```

---

# Lab 3 Cleanup

## Recommended

Do not delete Jenkins if you are continuing to Lab 4.

Keep:

```text
jenkins container
jenkins_home volume
jenkins network
```

Keep the GitHub application repository:

```text
jenkins-demo-app
```

We will use it in the next lab.

---

# Delete Only the Jenkins Lab 3 Job

From Jenkins:

```text
Dashboard
→ lab-03-git-checkout
→ Configure
→ Delete Project
```

Confirm deletion.

This removes the Jenkins job only.

It does not delete the GitHub repository.

---

# Delete the Local Application Repository

Only do this if you want to remove the local copy.

Go to:

```bash
cd /c/project
```

Remove:

```bash
rm -rf jenkins-demo-app
```

Verify:

```bash
ls
```

The GitHub repository will still exist online.

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
Install Jenkins
Run Jenkins in Docker
Create Jenkins administrator
Create Jenkins job
Run first build
```

Lab 2:

```text
Jobs
Builds
Build History
Workspace
Environment Variables
Build Parameters
Artifacts
```

Lab 3:

```text
Git
GitHub
Git Checkout
Jenkins Git Integration
Jenkins Workspace
GitHub Source Code
Build from Git
Code Changes
Git Push
Jenkins Rebuild
```

---

# Next Lab

## Lab 4 — Jenkins + GitHub Webhook

In Lab 3, you had to manually click:

```text
Build Now
```

In Lab 4, we will make GitHub notify Jenkins automatically.

The workflow will become:

```text
Developer
    |
    | git push
    v
GitHub
    |
    | Webhook
    v
Jenkins
    |
    v
Git Checkout
    |
    v
Build
    |
    v
Console Output
```

The goal is to make the pipeline react automatically when code is pushed to GitHub.

# End of Lab 3
