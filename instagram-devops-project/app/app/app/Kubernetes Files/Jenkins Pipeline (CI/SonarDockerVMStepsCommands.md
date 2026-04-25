in Jenkins = SonarQube Scanner = pluggin install

 SonarQube run
pooja@Ubuntu-VM:~$ docker run -d --name sonarqube \

http://<vm-ip>:9000
http://192.168.0.18:9000
docker ps -a:

pooja@Ubuntu-VM:~$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                         NAMES
fffddf87d5f8   sonarqube:lts   "/opt/sonarqube/dock…"   27 minutes ago   Up 27 minutes   0.0.0.0:9000->9000/tcp, [::]:9000->9000/tcp   sonarqube
pooja@Ubuntu-VM:~$

docker start SonarQube
docker stop SonarQube
===========================================================================================================================

Option 3: Run with restart policy (if recreating)

If you delete and run again, use:

docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  --restart unless-stopped \
  sonarqube:lts
🔍 Check status anytime
docker ps
docker start SonarQube
docker stop SonarQube

🔐 1. Generate Token in SonarQube
👉 Steps:

Open SonarQube:

http://192.168.0.20:9000
Login:
Username: admin
Password: admin
after change pwd: Poojabusa@09
Go to:
Top Right → My Account → Security
Under Tokens:
Name: jenkins-token
Click Generate
Copy the token

squ_dfb6003d0dfb32568aa6bbac4f1e1f97963949b6

http://192.168.0.20:9000/account/security

2. Setup Jenkins with SonarQube
✅ Step A: Install SonarQube Plugin

Open Jenkins:

http://192.168.0.20:8080
Go to:
Manage Jenkins → Plugins
Search: SonarQube Scanner
Install it
✅ Step B: Configure SonarQube in Jenkins
Go to:
Manage Jenkins → Configure System
Find SonarQube Servers
Add:
Name: SonarQube

Server URL:

http://192.168.0.20:9000
Token: (paste token you generated)
✅ Step C: Add Sonar Scanner
Go to:
Manage Jenkins → Global Tool Configuration
Under SonarQube Scanner:
Add → Name: SonarScanner
Install automatically ✅

.

🔐 Where to paste the token in Jenkins
✅ Step-by-step
1️⃣ Open Jenkins
http://192.168.0.20:8080
2️⃣ Go to:

👉 Manage Jenkins → Manage Credentials

3️⃣ Click:

👉 (global) → Add Credentials

4️⃣ Fill details like this:
Kind: Secret Text ✅
Scope: Global
Secret: 👉 Paste your SonarQube token here
ID: sonar-token (or any name)
Description: SonarQube Token

👉 Click OK

5️⃣ Now link it to SonarQube

Go to:

👉 Manage Jenkins → Configure System

Find SonarQube Servers

Click Add SonarQube

Fill:

Name: SonarQube

Server URL:

http://192.168.0.20:9000
Server authentication token:
👉 Select the credential you just created (sonar-token)
✅ Done 🎉

Now Jenkins can securely talk to SonarQube.

🔍 Quick Test

Run a pipeline → SonarQube stage
Then open:

vm server ip
http://192.168.0.20:9000


You should see your project analysis 
==================================================================

🚀 1. Run Nexus using Docker
docker run -d \
  --name nexus \
  -p 8081:8081 \
  --restart unless-stopped \
  sonatype/nexus3
🌐 2. Open Nexus UI

👉 Open in browser:

http://192.168.0.20:8081

⏳ First startup takes 2–5 minutes

🔐 3. Get Admin Password

Run:

docker exec nexus cat /nexus-data/admin.password

Login:
Username: admin
Password: (from above command)

👉 It will ask you to:

Change password
Disable anonymous access (choose as needed)
📦 4. Create Repository

Go to:
👉 Settings ⚙️ → Repositories → Create repository

✅ For Docker images

Choose:
👉 docker (hosted)

Fill:

Name: docker-hosted
HTTP Port: 8082
Enable: Deployment policy → Allow redeploy

Click Create

✅ For Maven (optional)

👉 Choose:

maven2 (hosted)
Name: maven-releases

⚙️ 5. Configure Docker to use Nexus

Edit Docker config:

sudo nano /etc/docker/daemon.json

Add:

{
  "insecure-registries": ["<your-ip>:8082"]
}

Restart Docker:

sudo systemctl restart docker

🔐 6. Login to Nexus (Docker)

docker login <your-ip>:8082

Username: admin
Password: your new password

🐳 7. Push Image to Nexus
Tag your image
docker tag instagram-ui 192.168.0.20:8082/instagram-ui
Push image
docker push 192.168.0.20:8082/instagram-ui
🔄 8. Pull image from Nexus
docker pull <your-ip>:8082/instagram-ui
⚙️ 9. Jenkins Integration

In Jenkins:

Add credentials

👉 Manage Jenkins → Credentials → Add

Kind: Username/Password
Username: admin
Password: Nexus password
ID: nexus-cred

=============================

🚀 Final Flow (Your pipeline)
Git pull
Build React app
SonarQube analysis ✅
Quality gate check ✅
Build Docker image
Push → DockerHub
Push → Nexus
Deploy → Kubernetes