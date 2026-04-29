### **TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION (INTRODUCTION TO JENKINS).**

In Project 8, we introduced the horizontal scalability concept, which allows us to add new Web Servers to our Tooling Website. You have successfully deployed a set-up with 2 Web Servers and a Load Balancer to distribute traffic between them. If it is just two or three servers – it is not a big deal to configure them manually. Imagine that you would need to repeat the same task repeatedly adding dozens or even hundreds of servers.

DevOps is about Agility and the speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is the Automation of routine tasks. 

In this project, we are going to start automating part of our routine tasks with a free and open-source automation server – [Jenkins](https://en.wikipedia.org/wiki/Jenkins_(software)). It is one of the most popular [CI/CD](https://en.wikipedia.org/wiki/CI/CD)tools, it was created by a former Sun Microsystems developer [Kohsuke Kawaguchi](https://en.wikipedia.org/wiki/Kohsuke_Kawaguchi) and the project originally had a name "Hudson".

According to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository. In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub `https://github.com/<yourname>/tooling` will automatically be updated to the Tooling Website.

###### **TASK**
Enhance the architecture prepared in [Project 8](https://github.com/theboyfromsalem/DevOps-Projects/tree/main/Project%208) by adding a Jenkins server, and configuring a job to automatically deploy source code changes from Git to the NFS server.

Here is what your updated architecture will look like upon completion of this project:

![alt text](<Images/1 (1).png>)

#### **STEP 1:** INSTALL THE JENKINS SERVER.

I launched a new Ubuntu EC2 instance and named it `Jenkins`.

Since Jenkins runs on Java, I installed JDK first:

```
sudo apt update
sudo apt install default-jdk-headless -y
```
![alt text](<Images/Screenshot 2026-04-26 075344.png>)

Add the Jenkins repository and Install it:

```
sudo rm -f /etc/apt/sources.list.d/jenkins.list
sudo rm -f /etc/apt/keyrings/jenkins-keyring.asc

sudo apt update
sudo apt install -y fontconfig openjdk-17-jre

sudo mkdir -p /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y jenkins
sudo systemctl enable --now jenkins
sudo systemctl status jenkins --no-pager
```
![alt text](<Images/Screenshot 2026-04-26 075846.png>)

![alt text](<Images/Screenshot 2026-04-26 075914.png>)

**OPEN PORT 8080:**

Jenkins runs on **TCP port 8080** by default. I added an inbound rule in the EC2 security group to allow traffic on that port so I could reach the Jenkins dashboard from my browser.

![alt text](<Images/Screenshot 2026-04-26 081352.png>)

**INITIAL SETUP**

Open Browser and access `http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`. You will be prompted to provide a default admin password.

![alt text](<Images/Screenshot 2026-04-26 082131.png>)

Jenkins asks for an initial admin password. Retrieve it from the server:

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

![alt text](<Images/Screenshot 2026-04-26 082840.png>)

Paste it in, chose `Install Suggested Plugins`, and create an admin user. Jenkins is ready!

![alt text](<Images/Screenshot 2026-04-26 083348.png>)

![alt text](<Images/Screenshot 2026-04-26 083821.png>)

#### **STEP 2:** CONNECT JENKINS TO RETRIEVE SOURCE CODES FROM GITHUB USING WEBHOOKS.

In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will be triggered by GitHub [webhooks](https://en.wikipedia.org/wiki/Webhook) and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

`http://<Jenkins-Public-IP>:8080/github-webhook/`
![alt text](<Images/Screenshot 2026-04-26 085242.png>)

![alt text](<Images/Screenshot 2026-04-26 085405.png>)

![alt text](<Images/Screenshot 2026-04-26 085500.png>)

![alt text](<Images/Screenshot 2026-04-26 090305.png>)

![alt text](<Images/Screenshot 2026-04-26 090355.png>)

**CREATE A FREESTYLE PROJECT IN JENKINS.**

In the Jenkins dashboard, Click on "**New Item**", name the project "**Pipeline**", and chose "**Freestyle Project**".

![alt text](<Images/Screenshot 2026-04-26 091541.png>)

![alt text](<Images/Screenshot 2026-04-26 091919.png>)

Under **Source Code Management**, Select Git and paste in your GitHub repository URL along with your credentials so Jenkins could access it.

![alt text](<Images/Screenshot 2026-04-26 093303.png>)

Save the configuration and let us try to run the build. For now, we can only do it manually. Click the "Build Now" button, if you have configured everything correctly, the build will be successful and you will see it under `#1`.

You can open the build and check in "Console Output" if it has run successfully. If so – congratulations! You have just made your very first Jenkins build! But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

![alt text](<Images/Screenshot 2026-04-26 093509.png>)

Click "**Configure**" on your job/project and add these two configurations

**a.** Configure "**triggers**":

![alt text](<Images/Screenshot 2026-04-26 093714.png>)

**b.** Configure "**Post-build Actions**":

![alt text](<Images/Screenshot 2026-04-26 094016.png>)

Now, go ahead and make some changes in any file in your GitHub repository (e.g. README.md file) and push the changes to the master branch. You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on the Jenkins server.

![alt text](<Images/Screenshot 2026-04-26 094845.png>)

The build artifacts are stored locally on the Jenkins server at:

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

![alt text](<Images/Screenshot 2026-04-26 095513.png>)

#### **STEP 3:** CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH. 

Having the files on the Jenkins server is not enough — they need to get to the NFS server so all the web servers can serve them. I used the **Publish Over SSH** plugin to make Jenkins push the files there automatically after every build.

**INSTALL THE PLUGIN**

In Jenkins: **Manage Jenkins** → **Manage Plugins** → **Available tab** → search for `Publish Over SSH` → installed it.

![alt text](<Images/Screenshot 2026-04-29 202147.png>)

![alt text](<Images/Screenshot 2026-04-29 202406.png>)

**CONFIGURE THE SSH CONNECTION**

In Jenkins: `Manage Jenkins` → `Configure System` → scrolled down to the `Publish over SSH section`.

Fill in:

- **Private Key** — the contents of the `.pem` file I use to SSH into the NFS server.

- **Name** — a label for the connection (e.g. `NFS-Server`)

- **Hostname** — the private IP address of the NFS server.

- **Username** — `ec2-user` (the default user on RHEL EC2 instances).

- **Remote Directory** — `/mnt/apps` (the NFS mount that all web servers read from).

![alt text](<Images/Screenshot 2026-04-29 205129.png>)

![alt text](<Images/Screenshot 2026-04-29 205721.png>)

**ADD THE POST-BUILD SSH TRANSFER**.
Back in the `pipeline` project configuration, Add another **Post-build Action**: **Send build artifacts over SSH**.

Configure it to:

- Use the `NFS-Server` connection I just set up.

- Transfer all files using the `**` pattern (everything the build produces).

- Send them to the `/mnt/apps` directory on the `NFS server`.


![alt text](<Images/Screenshot 2026-04-29 210043.png>)

Save and push another change to the GitHub README.md.

The build triggered automatically. In the console output, you should see the success logs.

![alt text](<Images/Screenshot 2026-04-29 213907.png>)

**VERIFY THE FILES LANDED ON THE NFS SERVER**.

- SSH into the NFS server and check:

```
cat /mnt/apps/README.md
```
![alt text](<Images/Screenshot 2026-04-29 222323.png>)
