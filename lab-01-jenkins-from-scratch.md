Part 1— Create Jenkins Network

Go back to the repository root if necessary:

cd ..

Create the Docker network:
```bash
docker network create jenkins
```
Check:
```bash
docker network ls
```
You should see:

jenkins

If Docker says the network already exists, that is okay.

Part 2 — Create Jenkins Volume

Create persistent storage for Jenkins:
```bash
docker volume create jenkins_home
```
Check:
```bash
docker volume ls
````
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

Part 3 — Start Jenkins

Run Jenkins:
```bash
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  --network jenkins \
  -p 8081:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```
Check the container:
```bash
docker ps
```
You should see a Jenkins container.

Example:

jenkins/jenkins:lts

Check the Jenkins logs:
```bash
docker logs jenkins
```
You can also follow the logs:
```bash
docker logs -f jenkins
```
Press:
```bash
Ctrl + C
```
to stop following the logs.

The container itself will continue running.

Part 4 — Get the Jenkins Initial Password

Run:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
Copy the password.

You will use it to unlock Jenkins.

Part 5 — Open Jenkins

Open your browser:
```bash
http://localhost:8081
```
You should see:

Unlock Jenkins

Paste the initial administrator password.

Part 6 — Install Jenkins Plugins

Select:

Install suggested plugins

Wait for the installation to complete.

Part 7 — Create Jenkins Administrator

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

Part 8 — Jenkins URL

Keep the Jenkins URL as:
```bash
http://localhost:8081/
```
Click:

Save and Finish

Then:

Start using Jenkins

You should now see the Jenkins Dashboard.

Part 9 — Create Your First Jenkins Job

From the Jenkins Dashboard:

New Item

Enter:

hello-jenkins

Select:

Freestyle project

Click:

OK

Part 10 — Add a Build Step

Find:

Build Steps

Select:

Execute shell

Add:
```bash
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
```
Click:

Save

Part 11 — Run the Jenkins Job

Click:
```bash
Build Now
```
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

Part 12 — Understand What Happened

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

Part 13 — Check Jenkins Container

From Git Bash:
```bash
docker ps
```
You should see:

jenkins

Inspect the container:
```bash
docker inspect jenkins
```
Check Jenkins logs:
```bash
docker logs jenkins
```
Part 14 — Verify Jenkins Volume

Run:
```bash
docker volume inspect jenkins_home
```
This confirms that Jenkins data is stored in the Docker volume.

Part 15 — Git Commit the Lab

Go to your repository root:
```bash
cd ~/OneDrive/Desktop/jenkins-zero-to-hero
```
Check status:
```bash
git status
```
Add the lab:
```bash
git add .
```
Commit:
```bash
git commit -m "Add Jenkins Lab 1 from scratch"
```
Push:
```bash
git push origin main
```
Refresh GitHub.

You should now see:

jenkins-zero-to-hero/
└── lab-01-jenkins-from-scratch/
    └── README.md

Part 16 — Lab 1 Cleanup

If you want to keep Jenkins for the next lab, do not run the cleanup commands.

Jenkins can continue running for Lab 2.

Stop Jenkins
```bash
docker stop jenkins
```
Remove Jenkins container
```bash
docker rm jenkins
```
Remove Jenkins volume

Warning: this deletes Jenkins jobs, plugins, configuration, build history, and credentials stored in this volume.
```bash
docker volume rm jenkins_home
```
Remove Jenkins network
```bash
docker network rm jenkins
```
Verify
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