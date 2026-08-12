Part 4 — Create Jenkins Network

Go back to the repository root if necessary:

cd ..

Create the Docker network:

docker network create jenkins

Check:

docker network ls

You should see:

jenkins

If Docker says the network already exists, that is okay.

Part 5 — Create Jenkins Volume

Create persistent storage for Jenkins:

docker volume create jenkins_home

Check:

docker volume ls

You should see:

jenkins_home

Why do we need a volume?

Jenkins stores its:

Jobs

Plugins

Credentials

Configuration

Build history

User information

inside:

/var/jenkins_home

The Docker volume keeps this data outside the container lifecycle.

Part 6 — Start Jenkins

Run Jenkins:

docker run -d \
  --name jenkins \
  --restart unless-stopped \
  --network jenkins \
  -p 8081:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

Check the container:

docker ps

You should see a Jenkins container.

Example:

jenkins/jenkins:lts

Check the Jenkins logs:

docker logs jenkins

You can also follow the logs:

docker logs -f jenkins

Press:

Ctrl + C

to stop following the logs.

The container itself will continue running.

Part 7 — Get the Jenkins Initial Password

Run:

docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

Copy the password.

You will use it to unlock Jenkins.

Part 8 — Open Jenkins

Open your browser:

http://localhost:8081

You should see:

Unlock Jenkins

Paste the initial administrator password.

Part 9 — Install Jenkins Plugins

Select:

Install suggested plugins

Wait for the installation to complete.

Part 10 — Create Jenkins Administrator

Create your Jenkins administrator.

Example:

Username:
admin

Password:
YOUR-PASSWORD

Full name:
Jenkins Admin

Email:
YOUR-EMAIL

Use your own password and email.

Do not commit passwords or secrets to GitHub.

Part 11 — Jenkins URL

Keep the Jenkins URL as:

http://localhost:8081/

Click:

Save and Finish

Then:

Start using Jenkins

You should now see the Jenkins Dashboard.

Part 12 — Create Your First Jenkins Job

From the Jenkins Dashboard:

New Item

Enter:

hello-jenkins

Select:

Freestyle project

Click:

OK

Part 13 — Add a Build Step

Find:

Build Steps

Select:

Execute shell

Add:

echo "================================"
echo "Hello from Jenkins!"
echo "================================"

echo "Jenkins is working."

echo "Current user:"
whoami

echo "Current directory:"
pwd

echo "Files:"
ls -la

echo "Build number:"
echo $BUILD_NUMBER

Click:

Save

Part 14 — Run the Jenkins Job

Click:

Build Now

You should see:

Build #1

Click:

#1

Then:

Console Output

You should see output similar to:

Hello from Jenkins!
Jenkins is working.
Current user:
jenkins

Current directory:
/var/jenkins_home/workspace/hello-jenkins

Build number:
1

The exact output can vary.

Part 15 — Understand What Happened

The basic Jenkins flow was:

Jenkins Dashboard
       |
       v
   New Item
       |
       v
 hello-jenkins
       |
       v
   Build Now
       |
       v
 Build #1
       |
       v
 Console Output

Jenkins created a workspace for the job and executed the shell commands.

Part 16 — Check Jenkins Container

From Git Bash:

docker ps

You should see:

jenkins

Inspect the container:

docker inspect jenkins

Check Jenkins logs:

docker logs jenkins

Part 17 — Verify Jenkins Volume

Run:

docker volume inspect jenkins_home

This confirms that Jenkins data is stored in the Docker volume.

Part 18 — Git Commit the Lab

Go to your repository root:

cd ~/OneDrive/Desktop/jenkins-zero-to-hero

Check status:

git status

Add the lab:

git add .

Commit:

git commit -m "Add Jenkins Lab 1 from scratch"

Push:

git push origin main

Refresh GitHub.

You should now see:

jenkins-zero-to-hero/
└── lab-01-jenkins-from-scratch/
    └── README.md

Part 19 — Lab 1 Cleanup

If you want to keep Jenkins for the next lab, do not run the cleanup commands.

Jenkins can continue running for Lab 2.

Stop Jenkins

docker stop jenkins

Remove Jenkins container

docker rm jenkins

Remove Jenkins volume

Warning: this deletes Jenkins jobs, plugins, configuration, build history, and credentials stored in this volume.

docker volume rm jenkins_home

Remove Jenkins network

docker network rm jenkins

Verify

docker ps -a

Check volumes:

docker volume ls

Check networks:

docker network ls

Important

If you want to continue to Lab 2, only leave the Jenkins container running.

You do not need to delete anything between labs.

The same Jenkins installation can be reused throughout the Zero-to-Hero course.

Jenkins Zero-to-Hero Roadmap

Lab 01
Jenkins from Scratch
        |
        v
Lab 02
Jenkins Jobs, Workspace & Environment Variables
        |
        v
Lab 03
Jenkins + Git
        |
        v
Lab 04
Jenkins + GitHub Webhook
        |
        v
Lab 05
Jenkins Pipeline + Jenkinsfile
        |
        v
Lab 06
Real CI Pipeline
        |
        v
Lab 07
Jenkins + Docker
        |
        v
Lab 08
Docker Hub
        |
        v
Lab 09
Jenkins + Kubernetes
        |
        v
Lab 10
Complete CI/CD Pipeline
        |
        v
Lab 11
Jenkins Credentials
        |
        v
Lab 12
Jenkins Agents
        |
        v
Lab 13
Shared Libraries
        |
        v
Lab 14
Jenkins Security
        |
        v
Lab 15
Production-style Jenkins
        |
        v
Final Project
GitHub → Jenkins → Docker → Kubernetes
