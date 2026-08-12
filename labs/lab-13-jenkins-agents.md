# Jenkins Zero to Hero — Lab 13

## Jenkins Controller and Agent

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
```

Until now, our Jenkins builds have been running on the Jenkins built-in node.

In this lab, we will learn how Jenkins can distribute work to other machines or containers called **agents**.

The architecture becomes:

```text
                    Jenkins Controller
                           |
          +----------------+----------------+
          |                                 |
          v                                 v
     Jenkins Agent 1                  Jenkins Agent 2
          |                                 |
          v                                 v
       Build                            Test
```

For this learning lab, we will create a Jenkins agent using a Docker container.

---

# Lab Objective

By the end of this lab, you will understand:

```text
Jenkins Controller
Jenkins Agent
Node
Executor
Label
Workspace
Agent Connection
Distributed Builds
Controller vs Agent
Pipeline Agent
```

You will also:

```text
1. Understand controller and agent architecture
2. Create a Docker-based Jenkins agent
3. Connect the agent to Jenkins
4. Assign a label
5. Run a job on the agent
6. Verify where the job actually ran
7. Use an agent in a Jenkins Pipeline
```

---

# Important

Jenkins terminology:

```text
Controller
```

The main Jenkins server.

```text
Agent
```

A machine or container that executes builds.

```text
Executor
```

A slot on an agent that can execute one build at a time.

```text
Label
```

A name used to select which agent should run a job.

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

Check the Jenkins image:

```bash
docker inspect jenkins --format "{{.Config.Image}}"
```

You should see something similar to:

```text
jenkins-docker:lts
```

Check Jenkins:

```text
http://localhost:8081
```

---

# Part 2 — Check the Current Jenkins Node

Open:

```text
Jenkins
→ Manage Jenkins
→ Nodes
```

Depending on your Jenkins version, you may see:

```text
Manage Jenkins
→ Nodes
```

or:

```text
Manage Jenkins
→ Manage Nodes and Clouds
```

You should see the built-in Jenkins node.

It may be named:

```text
Built-In Node
```

or:

```text
built-in
```

---

# Part 3 — Understand the Built-In Node

The built-in node is where Jenkins can execute builds when no separate agent is selected.

Our previous jobs used:

```groovy
agent any
```

This allowed Jenkins to select an available executor.

The architecture was:

```text
Jenkins Controller
       |
       v
Built-In Node
       |
       v
Build
```

---

# Part 4 — Check Executors

Open:

```text
Jenkins
→ Manage Jenkins
→ Nodes
→ Built-In Node
```

Look for:

```text
Executors
```

You may see:

```text
Number of executors:
2
```

The number can be different on your Jenkins installation.

An executor is a slot that can execute a build.

---

# Part 5 — Understand Multiple Executors

For example:

```text
Jenkins Agent
    |
    +---- Executor 1 → Build A
    |
    +---- Executor 2 → Build B
```

Two builds can run at the same time if two executors are available.

---

# Part 6 — Create a Docker Agent Directory

Open Git Bash.

Go to:

```bash
cd /c/project
```

Create:

```bash
mkdir jenkins-agent
```

Enter:

```bash
cd jenkins-agent
```

Check:

```bash
pwd
```

---

# Part 7 — Create Agent Dockerfile

Create:

```bash
touch Dockerfile
```

Open:

```bash
code Dockerfile
```

Paste:

```dockerfile
FROM eclipse-temurin:17-jdk-jammy

RUN apt-get update \
    && apt-get install -y curl git unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /home/jenkins

RUN useradd -m -d /home/jenkins -s /bin/bash jenkins

RUN chown -R jenkins:jenkins /home/jenkins

USER jenkins

WORKDIR /home/jenkins

CMD ["sleep", "infinity"]
```

Save.

---

# Part 8 — Understand the Agent Image

The agent image contains:

```text
Java
Git
Curl
Unzip
Linux shell
```

Jenkins agents need Java to run the Jenkins agent process.

Git is useful because our builds will work with Git repositories.

---

# Part 9 — Build the Agent Image

Run:

```bash
docker build -t jenkins-agent:1.0 .
```

Check:

```bash
docker images | grep jenkins-agent
```

Expected:

```text
jenkins-agent   1.0
```

---

# Part 10 — Test the Agent Container

Run:

```bash
docker run -d --name jenkins-agent-test jenkins-agent:1.0
```

Check:

```bash
docker ps --filter "name=jenkins-agent-test"
```

---

# Part 11 — Check Java Inside Agent

Run:

```bash
docker exec jenkins-agent-test java -version
```

You should see Java 17 information.

---

# Part 12 — Check Git Inside Agent

Run:

```bash
docker exec jenkins-agent-test git --version
```

Expected:

```text
git version ...
```

---

# Part 13 — Check Linux User

Run:

```bash
docker exec jenkins-agent-test whoami
```

Expected:

```text
jenkins
```

---

# Part 14 — Check Working Directory

Run:

```bash
docker exec jenkins-agent-test pwd
```

Expected:

```text
/home/jenkins
```

---

# Part 15 — Stop the Test Agent

We only used this container to test the image.

Stop it:

```bash
docker stop jenkins-agent-test
```

Remove it:

```bash
docker rm jenkins-agent-test
```

---

# Part 16 — Jenkins Agent Architecture

The architecture we want is:

```text
Docker Desktop
     |
     +-------------------------+
     |                         |
     v                         v
Jenkins Controller        Jenkins Agent
     |                         |
     |                         |
     +----------+--------------+
                |
                v
          Build Execution
```

---

# Part 17 — Install Docker Plugin Support

Open Jenkins:

```text
http://localhost:8081
```

Go to:

```text
Manage Jenkins
→ Plugins
```

Search for:

```text
Docker
```

For this lab, we will use the Docker Pipeline / Docker-related plugins needed by Jenkins to create or communicate with agents through Docker.

Install missing plugins if Jenkins reports they are required.

After installation, restart Jenkins only if Jenkins asks you to.

---

# Part 18 — Verify Docker Plugin

Go to:

```text
Manage Jenkins
→ Plugins
→ Installed plugins
```

Search:

```text
Docker
```

You should see the relevant Docker plugin components installed.

---

# Part 19 — Check Docker Connectivity From Jenkins

Run:

```bash
docker exec jenkins docker ps
```

Jenkins should be able to communicate with Docker.

This is important because the Docker-based agent will be created through Docker.

---

# Part 20 — Create a Cloud / Docker Configuration

Open:

```text
Jenkins
→ Manage Jenkins
```

Look for:

```text
Clouds
```

Depending on the Jenkins version, this may be under:

```text
Manage Jenkins
→ Clouds
```

or:

```text
Manage Nodes and Clouds
→ Configure Clouds
```

---

# Part 21 — Add Docker Cloud

Select:

```text
Add a new cloud
```

Choose:

```text
Docker
```

Enter a name:

```text
docker-cloud
```

Save temporarily.

---

# Part 22 — Configure Docker Server

In the Docker cloud configuration, locate the Docker server configuration.

Because Jenkins is running in Docker and the Docker socket is mounted, the Docker daemon is normally reachable through:

```text
unix:///var/run/docker.sock
```

Configure the Docker server connection using the available Docker cloud configuration fields.

Test the connection.

You should get a successful Docker connection.

---

# Part 23 — Add Docker Agent Template

Inside the Docker cloud configuration, add a Docker agent template.

Use:

```text
Docker Image:
jenkins-agent:1.0
```

Set a label:

```text
docker-agent
```

Set the remote working directory:

```text
/home/jenkins
```

Depending on your installed Docker plugin version, the exact UI names may be slightly different.

---

# Part 24 — Understand the Label

We created:

```text
docker-agent
```

This label allows Jenkins to select the Docker agent.

For example:

```groovy
agent {
    label 'docker-agent'
}
```

means:

```text
Run this Pipeline on an agent with the docker-agent label.
```

---

# Part 25 — Create a Simple Agent Pipeline

We will create a test job.

Open:

```text
Jenkins Dashboard
→ New Item
```

Enter:

```text
lab-13-agent-test
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

# Part 26 — Configure the Pipeline

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

    agent {
        label 'docker-agent'
    }

    stages {

        stage('Agent Information') {

            steps {

                echo '======================================'
                echo 'JENKINS AGENT TEST'
                echo '======================================'

                sh 'echo "Hostname:"'
                sh 'hostname'

                sh 'echo "User:"'
                sh 'whoami'

                sh 'echo "Working Directory:"'
                sh 'pwd'

                sh 'echo "Java Version:"'
                sh 'java -version'

                sh 'echo "Git Version:"'
                sh 'git --version'
            }
        }

        stage('Create File') {

            steps {

                sh 'echo "This file was created on the Jenkins agent." > agent.txt'

                sh 'cat agent.txt'
            }
        }
    }

    post {

        success {
            echo 'Agent pipeline completed successfully.'
        }

        failure {
            echo 'Agent pipeline failed.'
        }

        always {
            echo 'Agent pipeline finished.'
        }
    }
}
```

Click:

```text
Save
```

---

# Part 27 — Run the Agent Pipeline

Click:

```text
Build Now
```

Jenkins should try to provision the Docker agent.

The pipeline should then run on:

```text
docker-agent
```

---

# Part 28 — Check the Console Output

Open:

```text
Build #1
→ Console Output
```

You should see:

```text
JENKINS AGENT TEST
```

Then:

```text
Hostname:
```

Then:

```text
User:
jenkins
```

Then:

```text
Working Directory:
/home/jenkins
```

Then:

```text
Java Version:
```

Then:

```text
Git Version:
```

---

# Part 29 — Understand What Happened

The execution path was:

```text
Jenkins Controller
       |
       | Request agent
       v
Docker
       |
       v
jenkins-agent:1.0
       |
       v
docker-agent
       |
       v
Pipeline
```

The build did not execute on the original Jenkins controller container.

It executed on the agent.

---

# Part 30 — Check the Docker Agent Container

While a build is running, run:

```bash
docker ps
```

You may see the temporary agent container.

The exact name will depend on the Docker plugin.

If the build has already finished, the container may have been automatically removed.

---

# Part 31 — Compare Controller and Agent

Controller:

```text
Jenkins Dashboard
Jenkins Configuration
Jobs
Credentials
Pipeline Coordination
```

Agent:

```text
Build Commands
Shell Commands
Tests
Compilation
Docker Build
Package Creation
```

---

# Part 32 — Understand Controller and Agent

Conceptually:

```text
Controller
     |
     | Gives instructions
     v
Agent
     |
     | Executes commands
     v
Build
```

This allows Jenkins to distribute workloads.

---

# Part 33 — Understand Agent Workspace

The agent workspace may look like:

```text
/home/jenkins/workspace/<job-name>
```

Check the workspace from the pipeline:

```groovy
sh 'pwd'
```

The command should show the agent workspace.

---

# Part 34 — Add Workspace Verification

Open:

```text
lab-13-agent-test
→ Configure
```

In the pipeline add:

```groovy
stage('Workspace Test') {

    steps {

        sh 'pwd'

        sh 'ls -la'

        sh 'touch workspace-test.txt'

        sh 'ls -la workspace-test.txt'
    }
}
```

Save.

---

# Part 35 — Run Again

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
workspace-test.txt
```

This file was created on the agent workspace.

---

# Part 36 — Understand Executors

An agent has executors.

For example:

```text
Agent
 |
 +---- Executor 1
 |
 +---- Executor 2
```

If Executor 1 is busy:

```text
Build A → Executor 1
Build B → Executor 2
```

A third build may wait until an executor becomes available.

---

# Part 37 — Check Node Information

Open:

```text
Jenkins
→ Manage Jenkins
→ Nodes
```

You should see:

```text
Built-In Node
```

and your Docker agent if it is configured persistently.

---

# Part 38 — Understand Labels

Labels let you control where a build runs.

Example:

```text
docker-agent
```

Pipeline:

```groovy
agent {
    label 'docker-agent'
}
```

Another possible label:

```text
linux
```

Pipeline:

```groovy
agent {
    label 'linux'
}
```

---

# Part 39 — Create a Label-Based Pipeline

Create another Pipeline job:

```text
lab-13-label-test
```

Choose:

```text
Pipeline
```

Use:

```groovy
pipeline {

    agent {
        label 'docker-agent'
    }

    stages {

        stage('Label Test') {

            steps {

                echo 'Pipeline is running on docker-agent.'

                sh 'hostname'

                sh 'whoami'

                sh 'pwd'
            }
        }
    }
}
```

Save.

---

# Part 40 — Run Label Test

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
Pipeline is running on docker-agent.
```

---

# Part 41 — Use Agent in the Application Pipeline

Now we will modify the real CI/CD pipeline.

Go to:

```bash
cd /c/project/jenkins-demo-app
```

Open:

```bash
code Jenkinsfile
```

At the beginning, change:

```groovy
agent any
```

to:

```groovy
agent {
    label 'docker-agent'
}
```

Save.

---

# Part 42 — Check the Jenkinsfile

Run:

```bash
grep -n "agent" Jenkinsfile
```

Expected:

```text
agent {
    label 'docker-agent'
}
```

---

# Part 43 — Commit the Agent Change

Run:

```bash
git add Jenkinsfile
```

Commit:

```bash
git commit -m "Run CI/CD pipeline on Docker Jenkins agent"
```

Push:

```bash
git push origin main
```

---

# Part 44 — Run the Complete Pipeline

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
40.0
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

# Part 45 — Watch the Pipeline

The complete pipeline should now run on:

```text
docker-agent
```

The stages remain:

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
Build Information
Archive
```

---

# Part 46 — Verify the Agent in Console Output

Open:

```text
Console Output
```

Look for:

```text
Running on docker-agent
```

You should also see:

```text
/home/jenkins/workspace/...
```

This indicates that the Pipeline is running on the agent workspace.

---

# Part 47 — Verify Docker Access on the Agent

The agent image created earlier does not include the Docker CLI.

For the complete Docker pipeline to run on the agent, the agent image needs Docker CLI support.

We will therefore create a Docker-enabled agent image.

---

# Part 48 — Create Docker Agent Dockerfile

Go to:

```bash
cd /c/project/jenkins-agent
```

Open:

```bash
code Dockerfile
```

Replace it with:

```dockerfile
FROM eclipse-temurin:17-jdk-jammy

USER root

RUN apt-get update \
    && apt-get install -y \
       curl \
       git \
       unzip \
       ca-certificates \
    && install -m 0755 -d /etc/apt/keyrings \
    && curl -fsSL https://download.docker.com/linux/debian/gpg \
       -o /etc/apt/keyrings/docker.asc \
    && chmod a+r /etc/apt/keyrings/docker.asc \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
       | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt-get update \
    && apt-get install -y docker-ce-cli \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN useradd -m -d /home/jenkins -s /bin/bash jenkins

RUN mkdir -p /home/jenkins

RUN chown -R jenkins:jenkins /home/jenkins

USER jenkins

WORKDIR /home/jenkins

CMD ["sleep", "infinity"]
```

Save.

---

# Part 49 — Build Docker-Enabled Agent Image

Run:

```bash
docker build -t jenkins-agent:docker-1.0 .
```

Check:

```bash
docker images | grep jenkins-agent
```

You should see:

```text
jenkins-agent   docker-1.0
```

---

# Part 50 — Test Docker CLI in the Agent Image

Run:

```bash
docker run -d --name jenkins-agent-docker-test -v /var/run/docker.sock:/var/run/docker.sock jenkins-agent:docker-1.0
```

Check Docker:

```bash
docker exec jenkins-agent-docker-test docker --version
```

You should see a Docker version.

Test Docker connectivity:

```bash
docker exec jenkins-agent-docker-test docker ps
```

---

# Part 51 — Remove the Test Agent

Stop:

```bash
docker stop jenkins-agent-docker-test
```

Remove:

```bash
docker rm jenkins-agent-docker-test
```

---

# Part 52 — Update Docker Agent Template

In Jenkins, go to:

```text
Manage Jenkins
→ Clouds
→ docker-cloud
```

Edit the Docker agent template.

Change the Docker image from:

```text
jenkins-agent:1.0
```

to:

```text
jenkins-agent:docker-1.0
```

Keep the label:

```text
docker-agent
```

Save.

---

# Part 53 — Test the Docker Agent Again

Open:

```text
lab-13-agent-test
```

Run:

```text
Build Now
```

The console should show:

```text
docker
```

when you add this command:

```groovy
sh 'docker --version'
```

---

# Part 54 — Add Docker Test to the Agent Pipeline

Open:

```text
lab-13-agent-test
→ Configure
```

Add:

```groovy
stage('Docker Test') {

    steps {

        sh 'docker --version'

        sh 'docker ps'
    }
}
```

Save.

---

# Part 55 — Run Agent Docker Test

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
Docker version ...
```

and:

```text
CONTAINER ID
```

This proves the agent can access Docker.

---

# Part 56 — Understand Controller vs Docker Agent

Controller:

```text
Jenkins
```

Agent:

```text
jenkins-agent:docker-1.0
```

Docker:

```text
Docker Engine
```

The architecture is:

```text
Jenkins Controller
        |
        | Creates
        v
Docker Agent
        |
        | Docker CLI
        v
Docker Engine
```

---

# Part 57 — Run Complete CI/CD on the Agent

Open:

```text
lab-05-pipeline
```

Make sure:

```groovy
agent {
    label 'docker-agent'
}
```

is in the Jenkinsfile.

Run:

```text
Build with Parameters
```

Use:

```text
APP_VERSION:
41.0
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

# Part 58 — Verify Pipeline Agent

Open:

```text
Console Output
```

Verify:

```text
docker-agent
```

Then check the pipeline stages:

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
Build Information
Archive
```

---

# Part 59 — Verify Docker Image

Run:

```bash
docker images | grep jenkins-demo-app
```

You should see:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app   41.0
```

---

# Part 60 — Verify Docker Hub

Open:

```text
Docker Hub
→ jenkins-demo-app
```

Verify:

```text
41.0
```

---

# Part 61 — Verify Kubernetes

Run:

```bash
kubectl get deployment jenkins-demo-app -n jenkins-demo -o jsonpath="{.spec.template.spec.containers[0].image}"
```

Expected:

```text
YOUR-DOCKERHUB-USERNAME/jenkins-demo-app:41.0
```

---

# Part 62 — Check Kubernetes Pods

Run:

```bash
kubectl get pods -n jenkins-demo
```

They should be:

```text
Running
```

---

# Part 63 — Understand Distributed Builds

Before agents:

```text
Jenkins
   |
   +---- Build 1
   +---- Build 2
   +---- Build 3
```

With agents:

```text
Jenkins Controller
       |
       +---- Agent 1
       |       |
       |       +---- Build
       |
       +---- Agent 2
               |
               +---- Test
```

This allows Jenkins to distribute workload.

---

# Part 64 — Why Agents Matter

Agents allow you to have different build environments.

For example:

```text
linux-agent
    |
    v
Linux builds
```

```text
docker-agent
    |
    v
Docker builds
```

```text
windows-agent
    |
    v
Windows builds
```

```text
kubernetes-agent
    |
    v
Containerized builds
```

---

# Part 65 — Labels in Real Jenkins

Examples:

```text
docker
linux
java
node
python
kubernetes
windows
```

A pipeline can select a specific type:

```groovy
agent {
    label 'docker-agent'
}
```

---

# Part 66 — Understand Agent Workspace

When the pipeline runs on:

```text
docker-agent
```

the workspace belongs to that agent.

The controller coordinates the build, but the agent executes the shell commands.

---

# Part 67 — Check Job Console

Open:

```text
lab-05-pipeline
→ Latest Build
→ Console Output
```

Look for:

```text
Running on docker-agent
```

and:

```text
/home/jenkins/workspace/
```

---

# Part 68 — Test Agent Environment Variables

Create or modify:

```text
lab-13-agent-test
```

Use:

```groovy
pipeline {

    agent {
        label 'docker-agent'
    }

    stages {

        stage('Agent Environment') {

            steps {

                sh 'echo "JOB_NAME=$JOB_NAME"'

                sh 'echo "BUILD_NUMBER=$BUILD_NUMBER"'

                sh 'echo "NODE_NAME=$NODE_NAME"'

                sh 'echo "WORKSPACE=$WORKSPACE"'

                sh 'hostname'

                sh 'whoami'

                sh 'pwd'
            }
        }
    }
}
```

Save.

---

# Part 69 — Run Agent Environment Test

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
NODE_NAME
```

and:

```text
WORKSPACE
```

The node should correspond to your Docker agent.

---

# Part 70 — Understand `agent any`

When you use:

```groovy
agent any
```

Jenkins can run the Pipeline on any available eligible node.

---

# Part 71 — Understand `agent none`

You can also define:

```groovy
agent none
```

at the top level.

Then each stage can choose its own agent.

For example:

```groovy
pipeline {

    agent none

    stages {

        stage('Build') {

            agent {
                label 'docker-agent'
            }

            steps {
                sh 'echo "Build on Docker agent"'
            }
        }
    }
}
```

---

# Part 72 — Create a Stage-Specific Agent Example

Create a new Pipeline job:

```text
lab-13-stage-agent
```

Use:

```groovy
pipeline {

    agent none

    stages {

        stage('Controller or Default') {

            agent {
                label 'docker-agent'
            }

            steps {
                sh 'echo "Running on docker-agent"'
                sh 'hostname'
            }
        }

        stage('Second Agent Stage') {

            agent {
                label 'docker-agent'
            }

            steps {
                sh 'echo "Second stage running on docker-agent"'
                sh 'hostname'
            }
        }
    }
}
```

Save.

---

# Part 73 — Run Stage Agent Pipeline

Click:

```text
Build Now
```

Check:

```text
Console Output
```

You should see each stage running on the selected agent.

---

# Part 74 — Understand Agent Selection

The hierarchy is:

```text
pipeline
    |
    +---- agent any
    |
    +---- stage
            |
            +---- agent { label 'docker-agent' }
```

A stage-level agent can control where that stage executes.

---

# Part 75 — Save the Agent Lab Documentation

Go to the Jenkins documentation repository:

```bash
cd /c/project/jenkins-zero-to-hero
```

Go to:

```bash
cd labs
```

Create:

```bash
touch lab-13-jenkins-agents.md
```

Open:

```bash
code lab-13-jenkins-agents.md
```

Paste this complete lab into the file.

Save.

---

# Part 76 — Check Git Status

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
new file: labs/lab-13-jenkins-agents.md
```

---

# Part 77 — Add Lab 13

```bash
git add labs/lab-13-jenkins-agents.md
```

Check:

```bash
git status
```

---

# Part 78 — Commit Lab 13

```bash
git commit -m "Add Jenkins Lab 13 agents and distributed builds"
```

---

# Part 79 — Push Lab 13

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

# Part 80 — Verify Repository Structure

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
    └── lab-13-jenkins-agents.md
```

---

# Lab 13 Completion Checklist

```text
[ ] Jenkins controller understood
[ ] Jenkins built-in node understood
[ ] Executor understood
[ ] Agent understood
[ ] Jenkins agent Dockerfile created
[ ] Jenkins agent image built
[ ] Java verified on agent
[ ] Git verified on agent
[ ] Agent container tested
[ ] Docker cloud configured
[ ] Docker agent template configured
[ ] docker-agent label created
[ ] Agent pipeline created
[ ] Agent pipeline executed
[ ] Agent workspace verified
[ ] Agent environment variables checked
[ ] Docker-enabled agent image created
[ ] Docker CLI verified on agent
[ ] Docker access verified from agent
[ ] Complete CI/CD pipeline moved to agent
[ ] Docker image built on agent
[ ] Docker Hub push completed from agent
[ ] Kubernetes deployment completed from agent
[ ] Stage-specific agent understood
[ ] Distributed build concept understood
[ ] Lab 13 documentation saved
[ ] Lab 13 committed
[ ] Lab 13 pushed to GitHub
```

---

# Lab 13 Cleanup

## Delete Test Agent Jobs

From Jenkins:

```text
Dashboard
→ lab-13-agent-test
→ Configure
→ Delete Project
```

Delete:

```text
lab-13-label-test
```

Delete:

```text
lab-13-stage-agent
```

Keep:

```text
lab-05-pipeline
```

if you want to continue using the complete CI/CD pipeline.

---

# Remove Test Agent Images

Check:

```bash
docker images | grep jenkins-agent
```

Remove the basic test image if not needed:

```bash
docker rmi jenkins-agent:1.0
```

Remove the Docker-enabled agent image if not needed:

```bash
docker rmi jenkins-agent:docker-1.0
```

Only remove images you no longer need.

---

# Remove Agent Test Containers

Check:

```bash
docker ps -a --filter "name=jenkins-agent"
```

If an old test container exists:

```bash
docker rm -f jenkins-agent-test
```

If this exists:

```bash
docker rm -f jenkins-agent-docker-test
```

---

# Remove Docker Cloud Configuration

From Jenkins:

```text
Manage Jenkins
→ Clouds
→ docker-cloud
```

Delete the Docker cloud configuration if you no longer need dynamic agents.

---

# Keep the Agent Image for Future Labs

Recommended:

```text
jenkins-agent:docker-1.0
```

We may reuse the Docker agent in later labs.

---

# Complete Kubernetes Cleanup

If you want to remove the application:

```bash
kubectl delete deployment jenkins-demo-app -n jenkins-demo
```

Delete:

```bash
kubectl delete service jenkins-demo-service -n jenkins-demo
```

Or delete the entire namespace:

```bash
kubectl delete namespace jenkins-demo
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
Controller vs Agent
```

---

# Next Lab

## Lab 14 — Jenkins Shared Libraries

In the next lab, we will learn how to avoid repeating Jenkins Pipeline code across many projects.

The architecture will become:

```text
GitHub
   |
   v
Shared Library
   |
   +---- build()
   +---- test()
   +---- dockerBuild()
   +---- dockerPush()
   +---- deploy()
   |
   v
Project Jenkinsfile
```

We will learn:

```text
Shared Libraries
vars/
src/
Global Variables
Reusable Pipeline Functions
@Library
Centralized Jenkins Code
```

This is an important step toward production-style Jenkins.

# End of Lab 13
