# Jenkins Zero to Hero — Lab 15

## Jenkins Security

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
Lab 14 → Jenkins Shared Libraries
```

So far, we concentrated on making Jenkins work.

Now we will learn how to secure it.

The goal is to move from:

```text
Jenkins works
```

to:

```text
Jenkins is controlled
Jenkins is protected
Credentials are protected
Users have appropriate permissions
Jobs are not accessible to everyone
```

---

# Lab Objective

By the end of this lab, you will understand:

```text
Jenkins Authentication
Jenkins Authorization
Users
User Permissions
Roles
Matrix Authorization
Credentials Security
CSRF Protection
Agent Security
Login Security
Anonymous Access
Job Permissions
Security Best Practices
```

You will also:

```text
1. Create Jenkins users
2. Configure authentication
3. Configure authorization
4. Test user permissions
5. Restrict anonymous access
6. Understand CSRF protection
7. Review agent security
8. Review credential security
9. Verify Jenkins security settings
```

---

# Important

Security settings can lock you out of Jenkins if configured incorrectly.

For this lab:

```text
Keep your current admin session open.
```

Open a second browser window or private/incognito window for testing users.

Do not log out of your working administrator session until you confirm the new security settings work.

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

Open:

```text
http://localhost:8081
```

Log in with your current administrator account.

---

# Part 2 — Check Current Jenkins Version

Run:

```bash
docker exec jenkins java -version
```

Check the Jenkins image:

```bash
docker inspect jenkins --format "{{.Config.Image}}"
```

You should see something similar to:

```text
jenkins-docker:lts
```

---

# Part 3 — Check Jenkins Plugins

Open:

```text
Jenkins
→ Manage Jenkins
→ Plugins
→ Installed plugins
```

Search for:

```text
Matrix Authorization
```

Also search for:

```text
Role-based Authorization
```

Do not install both authorization strategies just for this lab.

We will use:

```text
Matrix-based security
```

for the basic hands-on exercise.

---

# Part 4 — Understand Authentication

Authentication answers:

```text
Who are you?
```

Example:

```text
admin
developer
tester
viewer
```

Authentication is the login process.

The flow is:

```text
Username
    +
Password
    |
    v
Jenkins
    |
    v
Authenticated User
```

---

# Part 5 — Understand Authorization

Authorization answers:

```text
What are you allowed to do?
```

For example:

```text
admin
    |
    +---- Configure Jenkins
    +---- Create Jobs
    +---- Delete Jobs
    +---- Manage Credentials

developer
    |
    +---- Build Jobs
    +---- View Jobs

viewer
    |
    +---- Read Jobs
```

---

# Part 6 — Check Current Security

Open:

```text
Manage Jenkins
→ Security
```

Depending on the Jenkins version, the security configuration may be under:

```text
Manage Jenkins
→ Security
```

or within:

```text
Manage Jenkins
→ Configure Global Security
```

Look at:

```text
Security Realm
```

and:

```text
Authorization
```

---

# Part 7 — Understand Security Realm

The Security Realm defines how Jenkins knows who users are.

Common approaches include:

```text
Jenkins own user database
LDAP
Active Directory
Single Sign-On
```

For this lab, we will use:

```text
Jenkins' own user database
```

---

# Part 8 — Create a Jenkins User

Open:

```text
Manage Jenkins
→ Users
```

Click:

```text
Create User
```

Create:

```text
Username:
developer
```

Password:

```text
Developer-Lab-123
```

Full name:

```text
DevOps Developer
```

Email:

```text
developer@example.com
```

Click:

```text
Create User
```

---

# Part 9 — Create a Viewer User

Create another user:

```text
viewer
```

Use:

```text
Password:
Viewer-Lab-123
```

Full name:

```text
Jenkins Viewer
```

Email:

```text
viewer@example.com
```

Click:

```text
Create User
```

---

# Part 10 — Verify Jenkins Users

Go to:

```text
Manage Jenkins
→ Users
```

You should now see:

```text
admin
developer
viewer
```

Your admin username may be different.

---

# Part 11 — Understand the Users

For this lab:

```text
admin
```

will be the administrator.

```text
developer
```

will represent a developer.

```text
viewer
```

will represent a read-only user.

---

# Part 12 — Keep the Administrator Session Open

Do not log out of your current administrator session.

Open a second browser window.

Open:

```text
http://localhost:8081
```

Use an incognito/private window if possible.

We will use it later to test:

```text
developer
viewer
```

---

# Part 13 — Configure Jenkins Security

Go to:

```text
Manage Jenkins
→ Security
```

Find:

```text
Security Realm
```

Select:

```text
Jenkins' own user database
```

Make sure:

```text
Allow users to sign up
```

is NOT enabled for the normal secured lab environment.

We created users manually.

---

# Part 14 — Understand User Signup

This option:

```text
Allow users to sign up
```

allows users to create their own Jenkins accounts.

For a controlled Jenkins server, we generally do not want anonymous visitors creating accounts.

So for this lab:

```text
Allow users to sign up
```

should be:

```text
Disabled
```

---

# Part 15 — Configure Authorization

Find:

```text
Authorization
```

Select:

```text
Matrix-based security
```

You should see a permission matrix.

---

# Part 16 — Understand Matrix Authorization

Matrix authorization gives permissions individually.

Example:

```text
User        Overall Read    Job Read    Job Build
-------------------------------------------------
admin          Yes             Yes          Yes
developer      Yes             Yes          Yes
viewer         Yes             Yes          No
```

The exact permissions we select will determine what each user can do.

---

# Part 17 — Configure Administrator Permissions

In the permission matrix, add your administrator user.

For example:

```text
admin
```

Select the permissions required for administration.

For the lab administrator, enable:

```text
Overall → Administer
```

This gives the administrator full Jenkins administration permissions.

Do not remove administrator access until you have confirmed another administrator account can log in.

---

# Part 18 — Configure Developer Permissions

Add:

```text
developer
```

Give the developer basic permissions such as:

```text
Overall → Read
```

For jobs, enable:

```text
Job → Read
Job → Build
Job → Cancel
```

Depending on the exact Jenkins version and permission matrix, the displayed permission names may be slightly different.

Do NOT give:

```text
Overall → Administer
```

to the developer.

---

# Part 19 — Configure Viewer Permissions

Add:

```text
viewer
```

Give:

```text
Overall → Read
```

and:

```text
Job → Read
```

Do not give:

```text
Job → Build
```

to the viewer.

---

# Part 20 — Review the Permission Matrix

Before saving, verify that the matrix roughly represents:

```text
admin
    |
    +---- Full administration

developer
    |
    +---- Read
    +---- Build
    +---- Cancel

viewer
    |
    +---- Read
```

The exact matrix will depend on your Jenkins plugin/version.

---

# Part 21 — Save Security Configuration

Click:

```text
Save
```

Keep the administrator browser session open.

---

# Part 22 — Test Developer Login

Open your second browser window:

```text
http://localhost:8081
```

Log in as:

```text
Username:
developer
```

Password:

```text
Developer-Lab-123
```

---

# Part 23 — Test Developer Access

The developer should be able to see Jenkins according to the permissions you assigned.

Open:

```text
Dashboard
```

Try opening:

```text
lab-05-pipeline
```

---

# Part 24 — Test Developer Build Permission

Open:

```text
lab-05-pipeline
```

Try:

```text
Build Now
```

or:

```text
Build with Parameters
```

The developer should be able to trigger the job because we granted:

```text
Job → Build
```

---

# Part 25 — Test Developer Configuration Access

Try:

```text
Configure
```

The developer should not have full Jenkins administration access.

Depending on the exact job-level permissions, the user may or may not have job configuration access.

For this lab, the important security principle is:

```text
Developer ≠ Jenkins Administrator
```

---

# Part 26 — Test Developer Jenkins Administration

Try opening:

```text
Manage Jenkins
```

The developer should not have administrative control.

If Jenkins refuses access, that is expected.

---

# Part 27 — Test Viewer Login

Log out of the developer account or use another private browser window.

Open:

```text
http://localhost:8081
```

Log in:

```text
Username:
viewer
```

Password:

```text
Viewer-Lab-123
```

---

# Part 28 — Test Viewer Access

Open:

```text
lab-05-pipeline
```

The viewer should be able to read the job.

---

# Part 29 — Test Viewer Build Permission

Try:

```text
Build Now
```

The viewer should NOT be allowed to build the job because we did not grant:

```text
Job → Build
```

This demonstrates authorization.

---

# Part 30 — Test Viewer Administration

Try:

```text
Manage Jenkins
```

The viewer should not be able to administer Jenkins.

---

# Part 31 — Understand the Security Difference

```text
Authentication
    |
    v
Who is the user?
```

Then:

```text
Authorization
    |
    v
What can the user do?
```

Example:

```text
developer
    |
    +---- Authentication → Successful
    |
    +---- Authorization → Build allowed
    |
    +---- Authorization → Administration denied
```

---

# Part 32 — Anonymous Access

A Jenkins server should not normally allow anonymous users to perform privileged operations.

In the authorization matrix, do not give anonymous users:

```text
Job Build
Job Configure
Job Delete
Overall Administer
Credentials permissions
```

---

# Part 33 — Test Anonymous Access

Log out of Jenkins.

Open:

```text
http://localhost:8081
```

If Jenkins redirects you to the login page, that is expected.

Try to open:

```text
http://localhost:8081/manage/
```

You should be required to authenticate.

---

# Part 34 — Understand Why Anonymous Access Matters

Without proper restrictions:

```text
Internet
   |
   v
Jenkins
   |
   +---- Build
   +---- Configure
   +---- Credentials
   +---- Administration
```

With authentication and authorization:

```text
Internet
   |
   v
Login
   |
   v
User
   |
   v
Permission Check
   |
   +---- Allowed
   |
   +---- Denied
```

---

# Part 35 — Understand CSRF Protection

CSRF means:

```text
Cross-Site Request Forgery
```

A CSRF attack attempts to trick an authenticated browser into performing an unwanted action.

Jenkins provides CSRF protection mechanisms.

---

# Part 36 — Check CSRF Protection

Open your administrator session:

```text
Manage Jenkins
→ Security
```

Look for:

```text
CSRF Protection
```

Make sure CSRF protection is enabled.

Depending on the Jenkins version, the UI may show a protection mechanism based on a crumb issuer.

---

# Part 37 — Understand Jenkins Crumb

A Jenkins crumb is a security token used to help verify that requests originate from an approved source rather than a forged request.

Conceptually:

```text
Browser
   |
   +---- Authentication
   |
   +---- Jenkins Crumb
   |
   v
Jenkins
```

This helps protect state-changing requests.

---

# Part 38 — Do Not Disable CSRF Protection

Do not disable CSRF protection simply because an API or webhook gives an error.

Instead:

```text
Check authentication
Check API usage
Check webhook endpoint
Check Jenkins logs
Check plugin documentation
```

---

# Part 39 — Review Credentials Security

Go to:

```text
Manage Jenkins
→ Credentials
→ Global
```

You should see credentials such as:

```text
dockerhub-credentials
```

and possibly:

```text
github-token
```

or:

```text
jenkins-ssh-lab
```

---

# Part 40 — Understand Credential Access

A developer should not automatically have permission to:

```text
Manage Jenkins
→ Credentials
```

Credential management should be restricted.

In a production environment, secrets should only be accessible to jobs/users that need them.

---

# Part 41 — Review Docker Socket Security

Our Jenkins container has:

```text
/var/run/docker.sock
```

mounted.

Check:

```bash
docker inspect jenkins --format "{{json .Mounts}}"
```

You should see the Docker socket mount.

---

# Part 42 — Understand Docker Socket Risk

The Docker socket gives Jenkins powerful control over Docker.

Conceptually:

```text
Jenkins
   |
   v
Docker Socket
   |
   v
Docker Engine
   |
   +---- Containers
   +---- Images
   +---- Volumes
   +---- Networks
```

A compromised Jenkins environment with Docker socket access can have significant control over the host Docker environment.

This is acceptable for this local learning lab, but it is a serious production security consideration.

---

# Part 43 — Review Kubernetes Access

Check Jenkins:

```bash
docker exec jenkins kubectl get nodes
```

Jenkins currently has Kubernetes access.

Check:

```bash
docker exec jenkins kubectl get namespaces
```

---

# Part 44 — Understand Kubernetes Credential Risk

If Jenkins can perform:

```text
kubectl create
kubectl delete
kubectl set image
```

then Jenkins has Kubernetes permissions.

In production, Jenkins should not automatically have unrestricted cluster-wide permissions.

A better design is:

```text
Jenkins
   |
   v
Dedicated ServiceAccount
   |
   v
Limited Role
   |
   v
Only required namespace
```

We will build a more controlled RBAC model in a later lab.

---

# Part 45 — Check Jenkins Credentials Permissions

As administrator, review:

```text
Manage Jenkins
→ Credentials
```

Make sure ordinary users do not have administrator-level access.

---

# Part 46 — Disable Signup

As administrator:

```text
Manage Jenkins
→ Security
```

Make sure:

```text
Allow users to sign up
```

is disabled.

---

# Part 47 — Review Authorization

Make sure:

```text
Authorization:
Matrix-based security
```

is still enabled.

Review:

```text
admin
developer
viewer
```

---

# Part 48 — Check Your Administrator Login

Before continuing, open a separate private browser window and verify your administrator login still works.

Use your actual administrator credentials.

Do not continue security changes until the administrator login is confirmed.

---

# Part 49 — Create a Security Test Job

Create:

```text
lab-15-security-test
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

# Part 50 — Add Security Test Pipeline

Use:

```groovy
pipeline {

    agent any

    stages {

        stage('Security Test') {

            steps {

                echo '======================================'
                echo 'JENKINS SECURITY LAB'
                echo '======================================'

                echo "User:"
                sh 'whoami'

                echo "Build User:"
                echo "${env.BUILD_USER_ID ?: 'system'}"

                echo "Job:"
                echo "$JOB_NAME"

                echo "Build:"
                echo "$BUILD_NUMBER"

                echo 'Security test completed.'
            }
        }
    }

    post {

        success {
            echo 'Security test completed successfully.'
        }

        failure {
            echo 'Security test failed.'
        }
    }
}
```

Save.

---

# Part 51 — Run Security Test

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
JENKINS SECURITY LAB
```

---

# Part 52 — Understand Build Identity

Jenkins may expose build-related environment information such as:

```text
JOB_NAME
BUILD_NUMBER
BUILD_URL
BUILD_TAG
```

These identify the build.

For more detailed user attribution, Jenkins may require an additional build-user plugin depending on the Jenkins configuration.

---

# Part 53 — Test Permission Boundaries

Use your:

```text
developer
```

account.

Try:

```text
Build Now
```

Expected:

```text
Allowed
```

Use:

```text
viewer
```

Try:

```text
Build Now
```

Expected:

```text
Denied
```

Use:

```text
admin
```

Try:

```text
Manage Jenkins
```

Expected:

```text
Allowed
```

---

# Part 54 — Understand Least Privilege

The principle is:

```text
Give users only the permissions they need.
```

Example:

```text
Viewer
    |
    +---- Read

Developer
    |
    +---- Read
    +---- Build

Administrator
    |
    +---- Full Administration
```

Do not give:

```text
Developer → Administer
```

unless there is a specific reason.

---

# Part 55 — Security Review Checklist

Review:

```text
Authentication enabled
Authorization enabled
Anonymous access restricted
User signup disabled
CSRF protection enabled
Credentials protected
Docker socket treated as privileged
Kubernetes access restricted
Administrator account protected
Users have least privilege
```

---

# Part 56 — Test the Docker Hub Pipeline

Now verify that security changes did not break your CI/CD pipeline.

Open:

```text
http://localhost:8081
```

Log in as:

```text
admin
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
60.0
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

# Part 57 — Verify Pipeline

The pipeline should execute:

```text
Checkout
Validate
Application Test
Docker Build
Docker Test
Docker Login
Docker Push
Kubernetes Deploy
Kubernetes Verify
Archive
```

---

# Part 58 — Verify Docker Hub

Open Docker Hub.

Check:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app
```

You should see:

```text
60.0
```

---

# Part 59 — Verify Kubernetes

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo
```

Check image:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:60.0
```

---

# Part 60 — Verify Kubernetes Pods

Run:

```bash
kubectl get pods -n jenkins-demo
```

The Pods should be:

```text
Running
```

---

# Part 61 — Understand Jenkins Security Architecture

Your secured Jenkins architecture is now:

```text
User
 |
 v
Authentication
 |
 v
Authorization
 |
 +----------------+
 |                |
 v                v
Allowed          Denied
 |
 v
Jenkins Job
 |
 v
Credentials
 |
 v
Docker / Kubernetes
```

---

# Part 62 — Security Architecture

```text
                    Jenkins
                       |
          +------------+------------+
          |                         |
          v                         v
   Authentication              Authorization
          |                         |
          v                         v
       Users                  Permissions
                                    |
             +----------------------+----------------+
             |                      |                |
             v                      v                v
            Jobs                Credentials       Admin
             |
             v
          Pipeline
             |
      +------+------+
      |             |
      v             v
    Docker      Kubernetes
```

---

# Part 63 — Why Jenkins Security Matters

Jenkins can control:

```text
Source Code
Docker
Docker Hub
Kubernetes
Credentials
Deployments
Builds
Infrastructure
```

A compromised Jenkins account can therefore be much more serious than a compromised ordinary application account.

This is why:

```text
Authentication
Authorization
Credentials
Least Privilege
CSRF Protection
Agent Security
```

matter.

---

# Part 64 — Check Jenkins Logs

As an administrator, you can inspect Jenkins logs:

```bash
docker logs jenkins --tail 100
```

For live logs:

```bash
docker logs -f jenkins
```

Press:

```text
Ctrl + C
```

to stop following the logs.

---

# Part 65 — Check Jenkins Container Permissions

Run:

```bash
docker inspect jenkins --format "{{.Config.User}}"
```

Your customized Jenkins image may run as:

```text
jenkins
```

or return an empty value if the image uses its configured default user.

---

# Part 66 — Review Mounted Volumes

Run:

```bash
docker inspect jenkins --format "{{json .Mounts}}"
```

You should see:

```text
jenkins_home
```

and:

```text
/var/run/docker.sock
```

Your Kubernetes kubeconfig may also be mounted depending on how you configured Lab 10.

---

# Part 67 — Understand Why Jenkins Data Must Be Protected

The Jenkins volume contains important data:

```text
Jobs
Plugins
Configuration
Credentials
Users
Build History
```

Do not expose:

```text
jenkins_home
```

to untrusted users.

---

# Part 68 — Check Jenkins Volume

Run:

```bash
docker volume inspect jenkins_home
```

---

# Part 69 — Do Not Delete Jenkins Data

Do NOT run this during the security lab:

```bash
docker volume rm jenkins_home
```

unless you intentionally want to destroy your Jenkins configuration.

---

# Part 70 — Create a Security Documentation File

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
touch lab-15-jenkins-security.md
```

Open:

```bash
code lab-15-jenkins-security.md
```

Paste this complete lab into the file.

Save.

---

# Part 71 — Check Git Status

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
new file: labs/lab-15-jenkins-security.md
```

---

# Part 72 — Add Lab 15

```bash
git add labs/lab-15-jenkins-security.md
```

Check:

```bash
git status
```

---

# Part 73 — Commit Lab 15

```bash
git commit -m "Add Jenkins Lab 15 security"
```

---

# Part 74 — Push Lab 15

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

# Part 75 — Verify Repository Structure

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
    ├── lab-14-jenkins-shared-libraries.md
    │
    └── lab-15-jenkins-security.md
```

---

# Lab 15 Completion Checklist

```text
[ ] Jenkins authentication understood
[ ] Jenkins authorization understood
[ ] Administrator user verified
[ ] Developer user created
[ ] Viewer user created
[ ] Security Realm reviewed
[ ] Jenkins own user database configured
[ ] User signup disabled
[ ] Matrix-based security configured
[ ] Administrator permissions verified
[ ] Developer permissions verified
[ ] Viewer permissions verified
[ ] Developer build access tested
[ ] Viewer build restriction tested
[ ] Anonymous access reviewed
[ ] CSRF protection reviewed
[ ] Jenkins credentials security reviewed
[ ] Docker socket security reviewed
[ ] Kubernetes access security reviewed
[ ] Least privilege understood
[ ] Security test pipeline created
[ ] Security test pipeline executed
[ ] Complete CI/CD pipeline tested after security changes
[ ] Docker Hub deployment verified
[ ] Kubernetes deployment verified
[ ] Jenkins logs reviewed
[ ] Jenkins volume reviewed
[ ] Lab 15 documentation saved
[ ] Lab 15 committed
[ ] Lab 15 pushed to GitHub
```

---

# Lab 15 Cleanup

## Delete the Security Test Job

From Jenkins:

```text
Dashboard
→ lab-15-security-test
→ Configure
→ Delete Project
```

---

# Delete the Test Users

Only delete these users if they were created specifically for this lab:

```text
developer
viewer
```

Go to:

```text
Manage Jenkins
→ Users
```

Open:

```text
developer
```

Delete the user if appropriate.

Then:

```text
viewer
```

Delete the user if appropriate.

Keep your administrator account.

---

# Remove Test Credentials

If you created test-only credentials in Lab 12 and have not already removed them, delete them:

```text
demo-secret-text
demo-username-password
github-token
jenkins-ssh-lab
```

Keep:

```text
dockerhub-credentials
```

if your CI/CD pipeline still needs it.

---

# Keep Security Settings

Do not turn security off after the lab.

Keep:

```text
Authentication enabled
Authorization enabled
User signup disabled
CSRF protection enabled
```

---

# Kubernetes Cleanup

If you want to remove the application:

```bash
kubectl delete deployment jenkins-demo-app -n jenkins-demo
```

Delete the Service:

```bash
kubectl delete service jenkins-demo-service -n jenkins-demo
```

Or delete the entire namespace:

```bash
kubectl delete namespace jenkins-demo
```

---

# Docker Image Cleanup

Check:

```bash
docker images | grep jenkins-demo-app
```

Remove a specific image if no longer needed:

```bash
docker rmi YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:60.0
```

---

# Complete Jenkins Cleanup

Only use these commands if you intentionally want to delete Jenkins completely.

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

Lab 15:

```text
Jenkins Authentication
Jenkins Authorization
Users
Permissions
Matrix-based Security
Least Privilege
Anonymous Access
CSRF Protection
Credential Security
Docker Socket Security
Kubernetes Access Security
Jenkins Security Review
```

---

# Next Lab

## Lab 16 — Final Production-Style Jenkins Project

This will be the final major lab.

We will combine everything:

```text
GitHub
   |
   | Push
   v
GitHub Webhook
   |
   v
Jenkins Controller
   |
   v
Jenkins Agent
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
Kubernetes / Minikube
   |
   v
Application
```

The final project will combine:

```text
Git
GitHub
Webhooks
Jenkins
Jenkinsfile
Shared Libraries
Jenkins Agents
Credentials
Docker
Docker Hub
Kubernetes
CI/CD
Security
Rollback
```

# End of Lab 15