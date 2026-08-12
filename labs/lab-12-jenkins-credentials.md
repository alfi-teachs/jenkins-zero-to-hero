# Jenkins Zero to Hero — Lab 12

## Jenkins Credentials and Secret Management

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
```

In previous labs, we used a Jenkins credential:

```text
dockerhub-credentials
```

In this lab, we will properly understand Jenkins credentials and how to use secrets without putting them inside the Jenkinsfile or GitHub repository.

The workflow becomes:

```text
Jenkins
   |
   +---- Credentials
   |       |
   |       +---- Docker Hub
   |       +---- Secret Text
   |       +---- Username + Password
   |
   v
Jenkins Pipeline
   |
   v
withCredentials
   |
   v
Application / Docker / API
```

---

# Lab Objective

By the end of this lab, you will understand:

```text
Jenkins Credentials
Credential ID
Username with Password
Secret Text
SSH Username with Private Key
Credentials Binding
withCredentials
Secret Masking
Environment Variables
Docker Hub Credentials
GitHub Token
Kubernetes Credentials
Credential Security
```

You will also learn an important rule:

```text
NEVER put passwords, tokens, API keys, or private keys inside Jenkinsfile or GitHub.
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

Check Docker access from Jenkins:

```bash
docker exec jenkins docker --version
```

Check Kubernetes access from Jenkins:

```bash
docker exec jenkins kubectl get nodes
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

You should have something similar to:

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

# Part 3 — Check Docker Hub Credential in Jenkins

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

Then:

```text
Global credentials
```

You should see:

```text
dockerhub-credentials
```

---

# Part 4 — Understand Jenkins Credential ID

The Jenkinsfile uses:

```text
dockerhub-credentials
```

This is the:

```text
Credential ID
```

The Jenkinsfile does NOT contain the actual password or token.

The relationship is:

```text
Jenkins Credential Store
          |
          v
dockerhub-credentials
          |
          v
Jenkinsfile
```

---

# Part 5 — Create a Secret Text Credential

We will create a test secret.

Do not use your real password.

Create a harmless test value such as:

```text
jenkins-secret-demo-123
```

In Jenkins:

```text
Manage Jenkins
→ Credentials
→ Global
→ Add Credentials
```

For:

```text
Kind
```

select:

```text
Secret text
```

For:

```text
Secret
```

enter:

```text
jenkins-secret-demo-123
```

For:

```text
ID
```

enter:

```text
demo-secret-text
```

For:

```text
Description
```

enter:

```text
Lab 12 secret text demonstration
```

Click:

```text
Create
```

---

# Part 6 — Verify the Secret Credential

Go to:

```text
Manage Jenkins
→ Credentials
→ Global
```

You should see:

```text
demo-secret-text
```

Do not expose or copy the secret anywhere.

---

# Part 7 — Create a Jenkins Credentials Test Job

Open:

```text
Jenkins Dashboard
```

Click:

```text
New Item
```

Enter:

```text
lab-12-credentials-test
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

# Part 8 — Add a Secret Text Pipeline

Scroll to:

```text
Pipeline
```

For:

```text
Definition
```

select:

```text
Pipeline script
```

Paste:

```groovy
pipeline {

    agent any

    stages {

        stage('Test Secret Credential') {

            steps {

                withCredentials([
                    string(
                        credentialsId: 'demo-secret-text',
                        variable: 'MY_SECRET'
                    )
                ]) {

                    sh '''
                        echo "Credential was loaded."
                        echo "Secret length:"
                        printf '%s' "$MY_SECRET" | wc -c
                    '''
                }
            }
        }
    }
}
```

Click:

```text
Save
```

---

# Part 9 — Run the Secret Test

Click:

```text
Build Now
```

Open:

```text
Build #1
```

Then:

```text
Console Output
```

You should see:

```text
Credential was loaded.
Secret length:
```

You should NOT see:

```text
jenkins-secret-demo-123
```

The important concept is:

```text
Secret
   |
   v
Jenkins Credential Store
   |
   v
withCredentials
   |
   v
Environment Variable
   |
   v
Pipeline
```

---

# Part 10 — Understand `withCredentials`

This:

```groovy
withCredentials([
    string(
        credentialsId: 'demo-secret-text',
        variable: 'MY_SECRET'
    )
])
```

means:

```text
Load credential:
demo-secret-text

Make it available temporarily as:
MY_SECRET
```

The secret is available inside that block.

---

# Part 11 — Test Secret Masking

Temporarily change the pipeline to:

```groovy
pipeline {

    agent any

    stages {

        stage('Test Masking') {

            steps {

                withCredentials([
                    string(
                        credentialsId: 'demo-secret-text',
                        variable: 'MY_SECRET'
                    )
                ]) {

                    sh '''
                        echo "The secret is:"
                        echo "$MY_SECRET"
                    '''
                }
            }
        }
    }
}
```

Save.

Run:

```text
Build Now
```

Open:

```text
Console Output
```

The secret should be masked rather than displayed as plain text.

Do not rely on masking as a reason to print secrets. The safest approach is:

```text
Do not print secrets.
```

---

# Part 12 — Restore the Safer Test

Replace the pipeline with:

```groovy
pipeline {

    agent any

    stages {

        stage('Test Secret Credential') {

            steps {

                withCredentials([
                    string(
                        credentialsId: 'demo-secret-text',
                        variable: 'MY_SECRET'
                    )
                ]) {

                    sh '''
                        echo "Credential loaded successfully."
                        echo "Secret length:"
                        printf '%s' "$MY_SECRET" | wc -c
                    '''
                }
            }
        }
    }
}
```

Save.

---

# Part 13 — Create a Username and Password Credential

Now we will create a second credential.

Go to:

```text
Manage Jenkins
→ Credentials
→ Global
→ Add Credentials
```

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
demo-user
```

For:

```text
Password
```

enter:

```text
demo-password-123
```

Use only this harmless lab value.

For:

```text
ID
```

enter:

```text
demo-username-password
```

For:

```text
Description
```

enter:

```text
Lab 12 username password demonstration
```

Click:

```text
Create
```

---

# Part 14 — Test Username and Password Credential

Open:

```text
lab-12-credentials-test
→ Configure
```

Replace the Pipeline script with:

```groovy
pipeline {

    agent any

    stages {

        stage('Test Username Password') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'demo-username-password',
                        usernameVariable: 'DEMO_USER',
                        passwordVariable: 'DEMO_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "Username credential loaded."

                        echo "Username length:"
                        printf '%s' "$DEMO_USER" | wc -c

                        echo "Password length:"
                        printf '%s' "$DEMO_PASSWORD" | wc -c
                    '''
                }
            }
        }
    }
}
```

Click:

```text
Save
```

---

# Part 15 — Run the Username Password Test

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
Username credential loaded.
Username length:
Password length:
```

The actual password should not appear.

---

# Part 16 — Understand Username and Password Binding

This:

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'demo-username-password',
        usernameVariable: 'DEMO_USER',
        passwordVariable: 'DEMO_PASSWORD'
    )
])
```

creates temporary variables:

```text
DEMO_USER
DEMO_PASSWORD
```

inside the credential block.

---

# Part 17 — Create a Docker Hub Credential

We already created this in earlier labs.

Verify:

```text
dockerhub-credentials
```

exists in:

```text
Jenkins
→ Manage Jenkins
→ Credentials
→ Global
```

If it does not exist, create:

```text
Kind:
Username with password
```

Username:

```text
YOUR-DOCKERHUB-USERNAME
```

Password:

```text
YOUR-DOCKERHUB-PERSONAL-ACCESS-TOKEN
```

ID:

```text
dockerhub-credentials
```

Description:

```text
Docker Hub credentials for Jenkins
```

Do not put the token into GitHub.

---

# Part 18 — Test Docker Hub Credential Safely

Open:

```text
lab-12-credentials-test
→ Configure
```

Replace the Pipeline script with:

```groovy
pipeline {

    agent any

    stages {

        stage('Test Docker Hub Credential') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {

                    sh '''
                        echo "Docker Hub username loaded."

                        echo "Username:"
                        echo "$DOCKER_USER"

                        echo "Token length:"
                        printf '%s' "$DOCKER_TOKEN" | wc -c
                    '''
                }
            }
        }
    }
}
```

Click:

```text
Save
```

---

# Part 19 — Run the Docker Credential Test

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
Docker Hub username loaded.
Username:
YOUR-DOCKERHUB-USERNAME

Token length:
```

The actual token should NOT appear.

---

# Part 20 — Understand Credentials in the Docker Pipeline

The complete flow is:

```text
Jenkins Credentials
        |
        v
dockerhub-credentials
        |
        v
withCredentials
        |
        +---- DOCKER_USER
        |
        +---- DOCKER_TOKEN
        |
        v
docker login
        |
        v
Docker Hub
```

---

# Part 21 — Create a GitHub Token Credential

For learning, we will create a credential representing a GitHub Personal Access Token.

Do not use your normal GitHub password.

If you do not already have a GitHub token, create one from GitHub:

```text
GitHub
→ Settings
→ Developer settings
→ Personal access tokens
```

Create a token appropriate for the repository operations you intend to automate.

For this lab, we will store it as:

```text
Secret text
```

---

# Part 22 — Create the GitHub Token Credential

Go to:

```text
Jenkins
→ Manage Jenkins
→ Credentials
→ Global
→ Add Credentials
```

Select:

```text
Secret text
```

For:

```text
Secret
```

paste your GitHub Personal Access Token.

For:

```text
ID
```

enter:

```text
github-token
```

For:

```text
Description
```

enter:

```text
GitHub token for Jenkins
```

Click:

```text
Create
```

---

# Part 23 — Verify GitHub Credential

Go to:

```text
Manage Jenkins
→ Credentials
→ Global
```

You should see:

```text
github-token
```

---

# Part 24 — Test GitHub Credential Without Printing It

Open:

```text
lab-12-credentials-test
→ Configure
```

Use:

```groovy
pipeline {

    agent any

    stages {

        stage('Test GitHub Credential') {

            steps {

                withCredentials([
                    string(
                        credentialsId: 'github-token',
                        variable: 'GITHUB_TOKEN'
                    )
                ]) {

                    sh '''
                        echo "GitHub credential loaded."
                        echo "Token length:"
                        printf '%s' "$GITHUB_TOKEN" | wc -c
                    '''
                }
            }
        }
    }
}
```

Save.

---

# Part 25 — Run GitHub Credential Test

Click:

```text
Build Now
```

Open:

```text
Console Output
```

Expected:

```text
GitHub credential loaded.
Token length:
```

The token itself should not appear.

---

# Part 26 — Understand Credential Types

Jenkins commonly supports credential types such as:

```text
Secret text
Username with password
SSH Username with private key
Certificate
Secret file
```

The type should match the way the secret is used.

---

# Part 27 — Secret Text Use Case

Use:

```text
Secret text
```

for things such as:

```text
API tokens
Access tokens
Webhook secrets
Single-value passwords
```

Example:

```text
github-token
```

---

# Part 28 — Username and Password Use Case

Use:

```text
Username with password
```

for:

```text
Docker Hub username + PAT
Registry username + password
Legacy application credentials
Basic authentication
```

Example:

```text
dockerhub-credentials
```

---

# Part 29 — SSH Key Use Case

Use:

```text
SSH Username with private key
```

for:

```text
Git over SSH
SSH servers
Remote Linux servers
Deployment servers
```

---

# Part 30 — Create an SSH Credential

We will create a real SSH key for lab practice.

Go to:

```bash
cd /c/project
```

Create a directory:

```bash
mkdir jenkins-ssh-lab
```

Enter:

```bash
cd jenkins-ssh-lab
```

Generate an SSH key:

```bash
ssh-keygen -t ed25519 -C "jenkins-lab@example.com" -f ./jenkins_lab_ed25519
```

When asked for a passphrase, you can enter one or leave it empty for this isolated lab key.

Check:

```bash
ls
```

You should see:

```text
jenkins_lab_ed25519
jenkins_lab_ed25519.pub
```

---

# Part 31 — View the Public Key

Run:

```bash
cat jenkins_lab_ed25519.pub
```

The output is safe to share as a public key.

Do not share:

```text
jenkins_lab_ed25519
```

The file without `.pub` is the private key.

---

# Part 32 — View the Private Key File

Do not paste or publish the following output anywhere.

You can verify that it exists:

```bash
test -f jenkins_lab_ed25519 && echo "Private key exists"
```

Expected:

```text
Private key exists
```

---

# Part 33 — Create the Jenkins SSH Credential

Open:

```text
Jenkins
→ Manage Jenkins
→ Credentials
→ Global
→ Add Credentials
```

Select:

```text
SSH Username with private key
```

For:

```text
Username
```

enter:

```text
jenkins
```

For:

```text
Private Key
```

select:

```text
Enter directly
```

Copy the contents of:

```text
jenkins_lab_ed25519
```

into the private key field.

For:

```text
ID
```

enter:

```text
jenkins-ssh-lab
```

For:

```text
Description
```

enter:

```text
Lab 12 SSH key
```

Click:

```text
Create
```

---

# Part 34 — Verify SSH Credential

Go to:

```text
Manage Jenkins
→ Credentials
→ Global
```

You should see:

```text
jenkins-ssh-lab
```

---

# Part 35 — Understand Private Key Security

This is important:

```text
Public Key
   |
   v
Can be shared

Private Key
   |
   v
Must remain secret
```

Never commit:

```text
jenkins_lab_ed25519
```

to GitHub.

---

# Part 36 — Create a Git Ignore Rule

Go back to:

```bash
cd /c/project/jenkins-ssh-lab
```

Create:

```bash
touch .gitignore
```

Open:

```bash
code .gitignore
```

Paste:

```text
jenkins_lab_ed25519
jenkins_lab_ed25519.pub
```

Save.

For this learning lab, however, we do not need to push this directory to GitHub.

---

# Part 37 — Check Jenkins Credential IDs

Your Jenkins credential store may now contain:

```text
dockerhub-credentials
demo-secret-text
demo-username-password
github-token
jenkins-ssh-lab
```

Your exact list may differ.

---

# Part 38 — Understand Credential Scope

Credentials can have different scopes and locations in Jenkins.

For our labs, we are using:

```text
Global
```

Global credentials are available to jobs that have access to that credential store.

In larger Jenkins installations, credentials can be organized more carefully.

---

# Part 39 — Return to the Application Repository

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

# Part 40 — Update the Complete CI/CD Pipeline

Open:

```bash
code Jenkinsfile
```

We will verify that Docker Hub authentication is handled through Jenkins credentials.

The important block should look like:

```groovy
stage('Docker Login') {

    steps {

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
    }
}
```

There should be no:

```text
dckr_pat_...
```

in the Jenkinsfile.

---

# Part 41 — Search the Repository for Token Patterns

Run:

```bash
grep -Rni "dckr_pat" . --exclude-dir=.git
```

Expected:

```text
```

There should be no Docker Hub Personal Access Token in the repository.

---

# Part 42 — Search for GitHub Token Patterns

Run:

```bash
grep -Rni "ghp_" . --exclude-dir=.git
```

Expected:

```text
```

There should be no GitHub Personal Access Token in the repository.

---

# Part 43 — Search for Common Password Variables

Run:

```bash
grep -RniE "password\s*=|token\s*=|secret\s*=" . --exclude-dir=.git
```

Review the results carefully.

References to Jenkins credential variables are fine.

Actual secrets are not.

---

# Part 44 — Good Jenkinsfile

Good:

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'dockerhub-credentials',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_TOKEN'
    )
]) {
}
```

Bad:

```groovy
environment {
    DOCKER_PASSWORD = 'mypassword'
}
```

Bad:

```groovy
sh 'docker login -u myuser -p mypassword'
```

Bad:

```groovy
DOCKER_TOKEN = 'dckr_pat_xxxxxxxxx'
```

---

# Part 45 — Run the Complete CI/CD Pipeline

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
30.0
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

# Part 46 — Check Docker Login Stage

Open:

```text
Console Output
```

Find:

```text
STAGE 6: DOCKER LOGIN
```

You should see:

```text
Docker Hub login completed.
```

You should NOT see your PAT.

---

# Part 47 — Check Docker Push

Find:

```text
STAGE 7: DOCKER PUSH
```

You should see:

```text
Docker image pushed
```

with:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:30.0
```

---

# Part 48 — Check Kubernetes Deployment

Find:

```text
STAGE 8: KUBERNETES DEPLOY
```

You should see:

```text
kubectl get nodes
```

and:

```text
kubectl rollout status
```

---

# Part 49 — Check Pipeline Success

At the end:

```text
CI/CD PIPELINE SUCCESS
```

---

# Part 50 — Verify Docker Hub

Open Docker Hub.

You should see:

```text
30.0
```

under:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

---

# Part 51 — Verify Kubernetes

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
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:30.0
```

---

# Part 52 — Verify Pods

Run:

```bash
kubectl get pods -n jenkins-demo
```

Pods should be:

```text
Running
```

---

# Part 53 — Check Jenkins Credentials Are Not Exposed

In Jenkins:

```text
Manage Jenkins
→ Credentials
→ Global
```

You can see credential IDs.

You cannot normally retrieve the original secret value from the credential listing.

This is the correct pattern:

```text
Credential ID
      |
      v
Jenkins Credential Store
      |
      v
Pipeline
      |
      v
Temporary Environment Variable
```

---

# Part 54 — Understand Secret Lifetime

Inside:

```groovy
withCredentials {
}
```

the variables are available to the steps inside the block.

After the block finishes, that credential binding is no longer available to later steps through that binding.

This limits unnecessary exposure.

---

# Part 55 — Do Not Print Secrets

Never use:

```bash
echo "$PASSWORD"
```

Never use:

```bash
echo "$TOKEN"
```

Never use:

```bash
cat ~/.ssh/id_ed25519
```

Never use:

```bash
env
```

when sensitive credentials are present and you do not know what the output contains.

---

# Part 56 — Safer Debugging

Instead of:

```bash
echo "$TOKEN"
```

use:

```bash
printf '%s' "$TOKEN" | wc -c
```

Instead of printing a password:

```bash
printf '%s' "$PASSWORD" | wc -c
```

This tells you that the variable exists without exposing the secret.

---

# Part 57 — Credential Flow in the Complete Pipeline

Your production-style flow is now:

```text
GitHub
   |
   v
Jenkins
   |
   +---- Credentials
   |        |
   |        +---- Docker Hub
   |        +---- GitHub
   |        +---- SSH
   |
   +---- Docker Build
   |
   +---- Docker Login
   |
   +---- Docker Push
   |
   +---- Kubernetes Deploy
   |
   v
Application
```

---

# Part 58 — Check Git Status

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

# Part 59 — Check Git History

Run:

```bash
git log --oneline --max-count=10
```

You should see your previous CI/CD commits.

---

# Part 60 — Save Lab 12 Documentation

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
touch lab-12-jenkins-credentials.md
```

Open:

```bash
code lab-12-jenkins-credentials.md
```

Paste this complete lab into the file.

Save.

---

# Part 61 — Check Git Status

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
new file: labs/lab-12-jenkins-credentials.md
```

---

# Part 62 — Add Lab 12

```bash
git add labs/lab-12-jenkins-credentials.md
```

Check:

```bash
git status
```

---

# Part 63 — Commit Lab 12

```bash
git commit -m "Add Jenkins Lab 12 credentials and secret management"
```

---

# Part 64 — Push Lab 12

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

# Part 65 — Verify Repository Structure

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
    ├── lab-10-jenkins-kubernetes.md
    │
    ├── lab-11-complete-cicd-pipeline.md
    │
    └── lab-12-jenkins-credentials.md
```

---

# Lab 12 Completion Checklist

```text
[ ] Jenkins credentials area located
[ ] Credential ID understood
[ ] Secret Text credential created
[ ] Secret Text tested
[ ] Secret masking observed
[ ] Username with Password credential created
[ ] Username with Password tested
[ ] Docker Hub credential verified
[ ] Docker Hub credential tested
[ ] GitHub token credential created
[ ] GitHub token tested
[ ] SSH key pair created
[ ] SSH credential created
[ ] Private key protected
[ ] Credential types understood
[ ] withCredentials understood
[ ] Secret variables understood
[ ] Secrets not stored in Jenkinsfile
[ ] Secrets not stored in GitHub
[ ] Docker Hub pipeline tested
[ ] Kubernetes deployment verified
[ ] Token patterns checked in repository
[ ] Git status clean
[ ] Lab 12 documentation saved
[ ] Lab 12 committed
[ ] Lab 12 pushed to GitHub
```

---

# Lab 12 Cleanup

## Delete the Test Jenkins Job

Open:

```text
Jenkins
→ Dashboard
→ lab-12-credentials-test
→ Configure
→ Delete Project
```

Confirm deletion.

---

# Delete Test Secret Text Credential

Go to:

```text
Jenkins
→ Manage Jenkins
→ Credentials
→ Global
```

Delete:

```text
demo-secret-text
```

---

# Delete Test Username Password Credential

Delete:

```text
demo-username-password
```

---

# Delete GitHub Token Credential

Delete:

```text
github-token
```

Only do this if you created it specifically for this lab.

---

# Delete SSH Credential

Delete:

```text
jenkins-ssh-lab
```

Only do this if you created it specifically for this lab.

---

# Docker Hub Credential

Keep:

```text
dockerhub-credentials
```

if you are continuing to use the complete CI/CD pipeline.

Do not delete it if Lab 11 and future labs depend on it.

---

# Delete Local SSH Lab Key

If you no longer need the SSH practice key:

```bash
cd /c/project/jenkins-ssh-lab
```

Remove the private key:

```bash
rm -f jenkins_lab_ed25519
```

Remove the public key:

```bash
rm -f jenkins_lab_ed25519.pub
```

Remove the Git ignore file:

```bash
rm -f .gitignore
```

Go back:

```bash
cd /c/project
```

Remove the directory:

```bash
rm -rf jenkins-ssh-lab
```

---

# Complete Jenkins Cleanup

Only use these commands if you intentionally want to remove Jenkins completely.

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

---

# Next Lab

## Lab 13 — Jenkins Agents

Until now, our builds have mostly run on the Jenkins built-in node.

In the next lab, we will learn the Jenkins controller/agent architecture.

The workflow will become:

```text
Jenkins Controller
       |
       +----------------+
       |                |
       v                v
   Agent 1           Agent 2
   Linux             Docker
       |                |
       v                v
    Build             Test
```

We will learn:

```text
Jenkins Controller
Jenkins Agent
Node
Executor
Labels
Agent Configuration
Agent Workspace
Controller vs Agent
Distributed Builds
```

# End of Lab 12
