# Jenkins Zero to Hero — Lab 4

## Jenkins + GitHub Webhook

This lab builds on:

```text
Lab 1 → Jenkins Installation
Lab 2 → Jenkins Jobs and Workspace
Lab 3 → Jenkins + Git + GitHub
```

In Lab 3, we manually started the Jenkins build using:

```text
Build Now
```

In Lab 4, GitHub will automatically notify Jenkins when code is pushed.

The workflow becomes:

```text
Developer
    |
    | git push
    v
GitHub
    |
    | Webhook
    v
Cloudflare Tunnel
    |
    v
Jenkins
    |
    v
Git Checkout
    |
    v
Build
```

---

# Lab Objective

By the end of this lab, you will understand:

```text
GitHub Webhooks
Jenkins GitHub Plugin
Jenkins Webhook Endpoint
Cloudflare Quick Tunnel
Automatic Jenkins Builds
Git Push → Jenkins Build
Webhook Testing
Webhook Troubleshooting
```

---

# Important

GitHub cannot send a webhook directly to:

```text
http://localhost:8081
```

GitHub requires a reachable webhook URL.

For this local lab, we will use a Cloudflare Quick Tunnel.

The tunnel will create a temporary public URL such as:

```text
https://something-random.trycloudflare.com
```

We will use:

```text
https://something-random.trycloudflare.com/github-webhook/
```

as the GitHub webhook URL.

The Quick Tunnel is for learning and testing only. The URL is temporary and can change when the tunnel is restarted.

---

# Prerequisites

Jenkins must already be running from the previous labs.

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

Check:

```bash
docker ps
```

Open Jenkins:

```text
http://localhost:8081
```

---

# Part 1 — Verify the Jenkins GitHub Plugin

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
Plugins
```

Search for:

```text
GitHub
```

Make sure the Jenkins GitHub plugin is installed.

The plugin provides GitHub integration and the webhook trigger used in this lab.

If it is not installed:

```text
Available plugins
→ Search GitHub
→ Install
```

After installation, restart Jenkins only if Jenkins asks you to.

Check Jenkins again:

```text
http://localhost:8081
```

---

# Part 2 — Verify the Application Repository

We will continue using the application repository from Lab 3:

```text
jenkins-demo-app
```

Go to the local repository:

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

You should have:

```text
README.md
app.sh
version.txt
```

---

# Part 3 — Check Git Status

Run:

```bash
git status
```

Expected:

```text
On branch main
nothing to commit, working tree clean
```

Your branch may show a different status if you have uncommitted changes.

---

# Part 4 — Check the GitHub Remote

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

# Part 5 — Verify Jenkins Job from Lab 3

Open Jenkins:

```text
http://localhost:8081
```

Open:

```text
lab-03-git-checkout
```

The job should still have:

```text
Source Code Management:
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

---

# Part 6 — Create the Jenkins Webhook Job

We will create a new job for Lab 4.

From the Jenkins dashboard:

```text
New Item
```

Enter:

```text
lab-04-github-webhook
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

# Part 7 — Configure GitHub Source Code

Find:

```text
Source Code Management
```

Select:

```text
Git
```

Repository URL:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app.git
```

For a public repository:

```text
Credentials:
- none -
```

Branch:

```text
*/main
```

---

# Part 8 — Enable the GitHub Webhook Trigger

Find:

```text
Build Triggers
```

Enable:

```text
GitHub hook trigger for GITScm polling
```

Do not select:

```text
Poll SCM
```

for this lab.

We want:

```text
GitHub
    |
    | webhook
    v
Jenkins
```

not:

```text
Jenkins
    |
    | repeatedly check GitHub
    v
GitHub
```

---

# Part 9 — Add the Jenkins Build Step

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
echo "JENKINS LAB 4"
echo "GITHUB WEBHOOK BUILD"
echo "======================================"

echo "Job Name:"
echo "$JOB_NAME"

echo "Build Number:"
echo "$BUILD_NUMBER"

echo "Build URL:"
echo "$BUILD_URL"

echo "Workspace:"
echo "$WORKSPACE"

echo "======================================"
echo "GIT INFORMATION"
echo "======================================"

git branch

git log --oneline -1

echo "======================================"
echo "APPLICATION VERSION"
echo "======================================"

cat version.txt

echo "======================================"
echo "RUNNING APPLICATION"
echo "======================================"

chmod +x app.sh

./app.sh

echo "======================================"
echo "WEBHOOK BUILD COMPLETED"
echo "======================================"
```

Click:

```text
Save
```

---

# Part 10 — Test the Job Manually First

Before testing the webhook, make sure the Jenkins job itself works.

Open:

```text
lab-04-github-webhook
```

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
JENKINS LAB 4
GITHUB WEBHOOK BUILD
```

You should also see your application version.

For example:

```text
2.0
```

The build must work manually before troubleshooting the webhook.

---

# Part 11 — Verify Jenkins Webhook Endpoint

The Jenkins GitHub plugin receives webhooks at:

```text
/github-webhook/
```

Because Jenkins is running locally on:

```text
http://localhost:8081
```

the local webhook endpoint is:

```text
http://localhost:8081/github-webhook/
```

GitHub cannot use this localhost URL directly.

We therefore need the Cloudflare tunnel.

---

# Part 12 — Create the Cloudflare Quick Tunnel

Open a **new Git Bash terminal**.

Do not close the Jenkins terminal.

Run:

```bash
docker run --rm \
  --name jenkins-webhook-tunnel \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate --url http://host.docker.internal:8081
```

Wait for the tunnel to start.

You should see a URL similar to:

```text
https://random-name.trycloudflare.com
```

The exact URL will be different.

---

# Part 13 — Important Cloudflare Tunnel Rule

Keep the Cloudflare terminal running.

Do not press:

```text
Ctrl + C
```

because that will stop the tunnel.

You now have:

```text
Internet
    |
    v
https://random-name.trycloudflare.com
    |
    v
Cloudflare Tunnel
    |
    v
http://host.docker.internal:8081
    |
    v
Jenkins
```

---

# Part 14 — Test the Cloudflare URL

Copy the `trycloudflare.com` URL printed by the tunnel.

For example:

```text
https://random-name.trycloudflare.com
```

Open it in your browser.

Then add:

```text
/github-webhook/
```

Your final URL will look like:

```text
https://random-name.trycloudflare.com/github-webhook/
```

You do not need to see the Jenkins dashboard at this path.

The important point is that the public URL can reach your Jenkins server.

---

# Part 15 — Add the Webhook to GitHub

Open the GitHub repository:

```text
jenkins-demo-app
```

Go to:

```text
Settings
```

Then:

```text
Webhooks
```

Click:

```text
Add webhook
```

---

# Part 16 — Enter the Payload URL

For:

```text
Payload URL
```

enter your Cloudflare URL followed by:

```text
/github-webhook/
```

Example:

```text
https://random-name.trycloudflare.com/github-webhook/
```

Do not use:

```text
http://localhost:8081/github-webhook/
```

Do not use:

```text
http://127.0.0.1:8081/github-webhook/
```

---

# Part 17 — Configure Content Type

For:

```text
Content type
```

select:

```text
application/json
```

---

# Part 18 — Webhook Secret

For this learning lab, you can leave:

```text
Secret
```

empty.

For production systems, use webhook authentication and a proper persistent tunnel or public Jenkins endpoint.

---

# Part 19 — Choose Which Events to Send

Select:

```text
Just the push event
```

This means GitHub will send a webhook when code is pushed.

Do not select every event for this lab.

---

# Part 20 — Enable the Webhook

Make sure:

```text
Active
```

is checked.

Click:

```text
Add webhook
```

---

# Part 21 — Check the GitHub Webhook

Go to:

```text
GitHub
→ Settings
→ Webhooks
```

You should see your new webhook.

The URL should contain:

```text
trycloudflare.com/github-webhook/
```

---

# Part 22 — Test the Webhook

Open the webhook.

Find:

```text
Recent Deliveries
```

Click the latest delivery.

GitHub should show the delivery details.

You want to see a successful response such as:

```text
200
```

or another successful 2xx response.

---

# Part 23 — Test Hook from GitHub

GitHub allows you to test the webhook.

Open:

```text
GitHub
→ Settings
→ Webhooks
→ Your webhook
```

Click:

```text
Test
```

or:

```text
Redeliver
```

depending on the GitHub UI.

Then check Jenkins.

---

# Part 24 — Check Jenkins for the Webhook

Open:

```text
lab-04-github-webhook
```

Look at:

```text
Build History
```

A successful webhook-triggered event should create a new build.

For example:

```text
#1
#2
```

---

# Part 25 — Make a Real Git Change

Now we will trigger Jenkins by pushing code.

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
2.0
```

to:

```text
3.0
```

Save.

---

# Part 26 — Change the Application

Open:

```bash
code app.sh
```

Change the script to:

```bash
#!/bin/sh

echo "======================================"
echo "JENKINS DEMO APPLICATION"
echo "======================================"

echo "Application: Jenkins Demo App"
echo "Environment: Jenkins CI"
echo "Version: 3.0"

echo "Application is running successfully."

echo "Webhook triggered build test."

echo "======================================"
```

Save.

---

# Part 27 — Test the Application Locally

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
Version: 3.0

Application is running successfully.

Webhook triggered build test.

======================================
```

Check:

```bash
cat version.txt
```

Expected:

```text
3.0
```

---

# Part 28 — Check Git Status

Run:

```bash
git status
```

Expected:

```text
modified: app.sh
modified: version.txt
```

---

# Part 29 — Commit the Change

Run:

```bash
git add app.sh version.txt
```

Commit:

```bash
git commit -m "Test Jenkins GitHub webhook"
```

---

# Part 30 — Push to GitHub

Run:

```bash
git push origin main
```

This is the important command.

The expected flow is now:

```text
git push
    |
    v
GitHub
    |
    | Webhook
    v
Cloudflare
    |
    v
Jenkins
    |
    v
Build
```

---

# Part 31 — Watch Jenkins

Immediately open:

```text
http://localhost:8081
```

Go to:

```text
lab-04-github-webhook
```

Check:

```text
Build History
```

A new build should appear.

For example:

```text
#2
```

or:

```text
#3
```

depending on how many builds you already created.

---

# Part 32 — Check the Jenkins Console

Open the new build.

Click:

```text
Console Output
```

You should see:

```text
JENKINS LAB 4
GITHUB WEBHOOK BUILD
```

You should also see:

```text
APPLICATION VERSION
3.0
```

And:

```text
Webhook triggered build test.
```

---

# Part 33 — Verify the Trigger

The important difference is:

Lab 3:

```text
git push
    |
    v
GitHub
    |
    X
Jenkins does nothing automatically
    |
    v
You click Build Now
```

Lab 4:

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
Automatic Build
```

---

# Part 34 — Verify the Jenkins Workspace

Run:

```bash
docker exec jenkins sh -c "ls -la /var/jenkins_home/workspace/lab-04-github-webhook"
```

Check the version:

```bash
docker exec jenkins sh -c "cat /var/jenkins_home/workspace/lab-04-github-webhook/version.txt"
```

Expected:

```text
3.0
```

Run the application from the Jenkins workspace:

```bash
docker exec jenkins sh -c "cd /var/jenkins_home/workspace/lab-04-github-webhook && chmod +x app.sh && ./app.sh"
```

---

# Part 35 — Check Git History in Jenkins Workspace

Run:

```bash
docker exec jenkins sh -c "cd /var/jenkins_home/workspace/lab-04-github-webhook && git log --oneline -3"
```

You should see the commit:

```text
Test Jenkins GitHub webhook
```

or the commit hash and message you created.

---

# Part 36 — Check GitHub Webhook Delivery

Go back to GitHub:

```text
jenkins-demo-app
→ Settings
→ Webhooks
```

Open the webhook.

Find:

```text
Recent Deliveries
```

The latest delivery should correspond to the Git push.

You can inspect:

```text
Request
Response
Status
```

---

# Part 37 — Test Another Push

Change the version again.

Open:

```bash
code version.txt
```

Change:

```text
3.0
```

to:

```text
4.0
```

Save.

---

# Part 38 — Commit Version 4.0

Run:

```bash
git add version.txt
```

Commit:

```bash
git commit -m "Update application version to 4.0"
```

Push:

```bash
git push origin main
```

---

# Part 39 — Check Jenkins Again

Open:

```text
http://localhost:8081
```

Open:

```text
lab-04-github-webhook
```

Check:

```text
Build History
```

A new build should appear automatically.

Open the new build:

```text
Console Output
```

You should now see:

```text
4.0
```

---

# Part 40 — Understand the Complete Jenkins Webhook Flow

```text
Developer
    |
    | 1. Change code
    |
    | 2. git add
    | 3. git commit
    | 4. git push
    |
    v
GitHub
    |
    | 5. Push event
    |
    v
GitHub Webhook
    |
    v
Cloudflare Quick Tunnel
    |
    v
Jenkins /github-webhook/
    |
    | 6. Trigger Git polling
    |
    v
Jenkins Git Checkout
    |
    v
Jenkins Workspace
    |
    v
Build Step
    |
    v
Console Output
```

---

# Part 41 — Why the Webhook Is Important

Without a webhook:

```text
Developer
    |
    v
GitHub
    |
    X
Jenkins does not know immediately
```

With a webhook:

```text
Developer
    |
    v
GitHub
    |
    | Push notification
    v
Jenkins
    |
    v
Build
```

This is one of the foundations of Continuous Integration.

---

# Part 42 — Check Cloudflare Tunnel

The Cloudflare tunnel terminal must still be running.

You should see something similar to:

```text
https://random-name.trycloudflare.com
```

Do not close that terminal while testing the webhook.

---

# Part 43 — If the Tunnel Stops

If you accidentally stop the tunnel with:

```text
Ctrl + C
```

the public URL will stop working.

Start a new tunnel:

```bash
docker run --rm \
  --name jenkins-webhook-tunnel \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate --url http://host.docker.internal:8081
```

A new temporary URL will be generated.

If the URL changes, update the GitHub webhook URL.

---

# Part 44 — Troubleshooting: Webhook Does Not Trigger Jenkins

Check that Jenkins is running:

```bash
docker ps
```

Check Jenkins directly:

```bash
docker exec jenkins sh -c "wget -qO- http://localhost:8080/login | head"
```

Check the tunnel:

```text
Cloudflare terminal
```

The tunnel must still be running.

---

# Part 45 — Troubleshooting: Verify the Webhook URL

Your GitHub webhook should look similar to:

```text
https://random-name.trycloudflare.com/github-webhook/
```

It should NOT be:

```text
http://localhost:8081/github-webhook/
```

It should NOT be:

```text
http://127.0.0.1:8081/github-webhook/
```

---

# Part 46 — Troubleshooting: Check Jenkins Trigger

Open:

```text
Jenkins
→ lab-04-github-webhook
→ Configure
```

Under:

```text
Build Triggers
```

make sure this is enabled:

```text
GitHub hook trigger for GITScm polling
```

Click:

```text
Save
```

---

# Part 47 — Troubleshooting: Check GitHub Delivery

Go to:

```text
GitHub
→ jenkins-demo-app
→ Settings
→ Webhooks
→ Your webhook
```

Check:

```text
Recent Deliveries
```

Look at the response status.

A successful HTTP response should be in the 2xx range.

---

# Part 48 — Troubleshooting: Test the Webhook Again

From GitHub:

```text
Settings
→ Webhooks
→ Your webhook
→ Recent Deliveries
```

Use:

```text
Redeliver
```

Then return to Jenkins.

Check:

```text
Build History
```

---

# Part 49 — Troubleshooting: Check Jenkins Logs

Open Git Bash:

```bash
docker logs jenkins --tail 100
```

To watch the logs:

```bash
docker logs -f jenkins
```

Press:

```text
Ctrl + C
```

to stop watching the logs.

This does not stop Jenkins.

---

# Part 50 — Verify the GitHub Plugin

From Jenkins:

```text
Manage Jenkins
→ Plugins
→ Installed plugins
```

Search:

```text
GitHub
```

Make sure the GitHub integration plugin is installed.

---

# Part 51 — Check Jenkins Job Configuration

Verify:

```text
Job:
lab-04-github-webhook
```

Repository:

```text
https://github.com/YOUR-GITHUB-USERNAME/jenkins-demo-app.git
```

Branch:

```text
*/main
```

Build Trigger:

```text
GitHub hook trigger for GITScm polling
```

Build Step:

```text
Execute shell
```

---

# Part 52 — Save Lab 4 Documentation

Go to the Jenkins training repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go to labs:

```bash
cd labs
```

Open:

```bash
code lab-04-jenkins-webhooks.md
```

Paste this complete lab into the file.

Save it.

---

# Part 53 — Check Git Status

Go to the repository root:

```bash
cd /c/project/jenkins-zero-to-hero
```

Run:

```bash
git status
```

You should see:

```text
modified: labs/lab-04-jenkins-webhooks.md
```

or:

```text
new file: labs/lab-04-jenkins-webhooks.md
```

---

# Part 54 — Add Lab 4

```bash
git add labs/lab-04-jenkins-webhooks.md
```

Check:

```bash
git status
```

---

# Part 55 — Commit Lab 4

```bash
git commit -m "Add Jenkins Lab 4 GitHub webhook"
```

---

# Part 56 — Push Lab 4 to GitHub

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

# Part 57 — Verify the Jenkins Repository

Your documentation repository should now look like:

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
    └── lab-04-jenkins-webhooks.md
```

---

# Lab 4 Completion Checklist

```text
[ ] Jenkins GitHub plugin verified
[ ] lab-04-github-webhook Jenkins job created
[ ] GitHub repository configured
[ ] main branch configured
[ ] GitHub hook trigger enabled
[ ] Jenkins build tested manually
[ ] Cloudflare Quick Tunnel started
[ ] Public Cloudflare URL obtained
[ ] GitHub webhook created
[ ] Payload URL configured
[ ] Content type set to application/json
[ ] Push event selected
[ ] Webhook enabled
[ ] GitHub webhook tested
[ ] Webhook delivery checked
[ ] GitHub push performed
[ ] Jenkins automatically triggered
[ ] Jenkins workspace updated
[ ] Application version verified
[ ] Second webhook test completed
[ ] Jenkins logs checked
[ ] Lab 4 documentation saved
[ ] Lab 4 committed
[ ] Lab 4 pushed to GitHub
```

---

# Lab 4 Cleanup

## Recommended

If you are continuing to Lab 5, keep Jenkins running.

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

# Stop Only the Webhook Tunnel

When you are finished testing Lab 4, press:

```text
Ctrl + C
```

in the Cloudflare tunnel terminal.

This stops only the temporary tunnel.

Jenkins continues running.

---

# Delete the Jenkins Lab 4 Job

From Jenkins:

```text
Dashboard
→ lab-04-github-webhook
→ Configure
→ Delete Project
```

Confirm deletion if you do not need the job.

---

# Delete the GitHub Webhook

Go to:

```text
GitHub
→ jenkins-demo-app
→ Settings
→ Webhooks
```

Open the Jenkins webhook.

Scroll down.

Click:

```text
Delete webhook
```

Confirm.

---

# Complete Jenkins Cleanup

Only do this when you want to completely remove Jenkins.

Stop Jenkins:

```bash
docker stop jenkins
```

Remove Jenkins:

```bash
docker rm jenkins
```

Remove Jenkins data:

```bash
docker volume rm jenkins_home
```

Remove the network:

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

Verify network:

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
Building from GitHub
```

Lab 4:

```text
GitHub Webhooks
Jenkins Webhook Endpoint
Cloudflare Quick Tunnel
Automatic Builds
Git Push → Webhook → Jenkins
Webhook Testing
Webhook Troubleshooting
```

---

# Next Lab

## Lab 5 — Jenkins Pipeline

Up to now, we used:

```text
Freestyle Project
```

In Lab 5, we will start using:

```text
Pipeline
```

The workflow will become:

```text
GitHub
   |
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

We will start writing Jenkins Pipeline code and introduce the:

```text
Jenkinsfile
```

# End of Lab 4