# Node.js App Deployment using GitHub, Jenkins, CI/CD, and AWS EC2
### This project demonstrates deploying a **Node.js application** on an **AWS EC2 instance** using a **Jenkins CI/CD pipeline** integrated with GitHub.It includes **webhooks, plugins, credentials, Jenkinsfile**, and an **automated deployment workflow**.  
##  Project Overview  
- **Source Code**: GitHub repository (Node.js app).  
- **CI/CD Tool**: Jenkins (with pipeline + plugins).  
- **Deployment Target**: AWS EC2 instance.  
- **Automation**: Triggered by GitHub Webhook.  
- **Security**: Jenkins credentials for GitHub & EC2 SSH.  

## Tools & Technologies  
- **Node.js** → Application backend  
- **GitHub** → Version control  
- **Jenkins** → CI/CD automation  
- **AWS EC2** → Server for hosting Node.js app  
- **Jenkins Plugins** → Git, Pipeline, SSH Agent, Github plugin  
- **Webhook** → Automates Jenkins builds from GitHub commits  
---
## Steps to Deploy
## Step 1) Set up Jenkins 
### Launch AWS EC2 Instance
- Go to **AWS Console** 
-  Launch EC2 
    - AMI - Ubuntu
    - Key - .pem key
    - In security group Open Ports :  `22 (SSH)`, `8080 (Jenkins)`, `80 (HTTP)`
    - Connect via SSH: 
    ```bash
    ssh -i <key> ubuntu@<EC2_PUBLIC_IP>
    ```
### Install Dependencies on EC2
```bash
# Update the server
sudo apt update && sudo apt upgrade -y

#install java
sudo apt install openjdk-17-jdk -y

#Install jenkins

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update

sudo apt-get install jenkins
```
- Copy the < `.pem` > to the Jenkins server to connect the node app server
```bash
scp -i <key> <key> ubuntu@<EC2-Public-IP>:/home/ubuntu/
``` 
- Copy `Public-IP of Jenkins `
 ![alt text](<Images/Screenshot 2025-08-16 000629.png>)
- Open Browser and check Jenkins is working or not
- Access Jenkins: <EC2_PUBLIC_IP>:8080
- Configure Jenkins
- Create Admin User
 ![alt text](<Images/Screenshot 2025-08-16 000719.png>)

### Install Required Jenkins Plugins
- Go to Jenkins, `Settings`
- Go to `Manage Jenkins → plugins.`
 ![alt text](<Images/Screenshot 2025-08-16 000932.png>)
- Click on `Plugins`
![alt text](<Images/Screenshot 2025-08-16 003025.png>)
- Click on `Available plugins` and search for Plugins
  - Git Plugin
  - Pipeline Plugin
  - SSH Agent Plugin
  - GitHub Plugin
![alt text](<Images/Screenshot 2025-08-16 001041.png>)
![alt text](<Images/Screenshot 2025-08-16 001124.png>)
![alt text](<Images/Screenshot 2025-08-16 001151.png>)
- Install all this plugins

### Add Credentials in Jenkins
- Go to `Manage Jenkins → Credentials.`
![alt text](<Images/Screenshot 2025-08-16 003054.png>)
- Click on ` add credentials `
![alt text](<Images/Screenshot 2025-08-16 003138.png>)
- In that create new credential
>In kind - SSH Username with Private key 
![alt text](<Images/Screenshot 2025-08-16 003243.png>)
> Scope - Default given <br>
> ID - Name for credential<br>
> Description - < ><br>
> Username - <AMIs used in server> ubuntu<br>
![alt text](<Images/Screenshot 2025-08-16 003345.png>)
>In Private key,
![alt text](<Images/Screenshot 2025-08-16 003543.png>)
> click on add key and copy the key from git terminal and Paste in it and click on create
![alt text](<Images/Screenshot 2025-08-16 003448.png>)
- Finally Credential is created
![alt text](<Images/Screenshot 2025-08-16 003602.png>)
## Step 2) Create Node app server
### Launch AWS EC2 Instance
- Go to **AWS Console** 
-  Launch EC2 
    - AMI - Ubuntu
    - Key - .pem key
    - In security group Open Ports :  `22 (SSH)`, `3000 (Node js)`, `80 (HTTP)`
    - Connect via SSH: 
    ```bash
    ssh -i <key> ubuntu@<EC2_PUBLIC_IP>
    ```
 ![alt text](<Images/Screenshot 2025-08-16 003835.png>)
 ![alt text](<Images/Screenshot 2025-08-16 003855.png>)
### Set up Node app server
```bash 
# Update the server
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_lts.x | bash -
sudo apt install -y nodejs

# Install npm
sudo apt install npm

# Install PM2 globally for Jenkins user
npm install -g pm2

# Verify installation
node -v
npm -v
pm2 -v
```
![alt text](<Images/Screenshot 2025-08-16 004842.png>)
## Step 3) Setup GitHub Repository
- Create one Repo on Github 
- Push your Node.js app to GitHub repo which we created.
 ![alt text](<Images/Screenshot 2025-08-16 005550.png>)
- Add pipeline script from `Jenkinsfile` in GitHub.
```bash
# The whole jenkinsfile is written in jenkinsfile  
environment {
        SERVER_IP      = '3.93.184.84' # node app server public ip
        SSH_CREDENTIAL = 'node-app-key'
        REPO_URL       = 'https://github.com/iamonkarlad/node-js-app.git' # github repo of node app
        BRANCH         = 'master'   # check branch 
        REMOTE_USER    = 'ubuntu'
        REMOTE_PATH    = '/home/ubuntu/node-app' #path of app to store
    }
```
- [Jenkinsfile](jenkinsfile) - click to get whole jenkinsfile
- check that all information is correct 
![alt text](<Images/Screenshot 2025-08-16 005754.png>)
- Push the jenkinsfile to github repo

## Step 4) Create Job 
- Go to `Jenkins → New Item → Pipeline. `

 ![alt text](<Images/Screenshot 2025-08-16 005841.png>)
- Give `item name`
- Select item type - `Pipeline`
 ![alt text](<Images/Screenshot 2025-08-16 005908.png>)

### Configure job Pipeline
- Configure the created ` Item `
- Kept changes default up to `Trigger` section
- In `Triggers` section, ***Check in*** the `Github hook trigger for GITScm polling  `, for the Webhooks 
![alt text](<Images/Screenshot 2025-08-16 005951.png>)
- In `Pipeline` section 
- Select `Pipeline script from SCM.`
![alt text](<Images/Screenshot 2025-08-16 010013.png>)
- In that `SCM`
- Select `GIT`
![alt text](<Images/Screenshot 2025-08-16 010026.png>)
- In `Repository URL` copy the URL from GitHub 
![alt text](<Images/Screenshot 2025-08-16 005608.png>)
- And paste it on Repository URL
![alt text](<Images/Screenshot 2025-08-16 010128.png>)
- Make other changes default
- In `Branch Specifier` add branch, check the branch of created repo from github 
- `*/master`
![alt text](<Images/Screenshot 2025-08-16 010157.png>)
- other changes kept Default
- In `Script path`, add the jenkinsfile path means file name where the file is written, as it is given on Github, copy it and paste the jenkinsfile name
- `jenkinsfile`
- And save the configuration
![alt text](<Images/Screenshot 2025-08-16 010217.png>)
 ## Step 5) Run Job
- Click on `Build Now`, wait for seconds
- Job is been executing
 ![alt text](<Images/Screenshot 2025-08-16 010329.png>)
- If there is Green tick, then it is successfull build
- If there is Red cross, then it is unsuccessfull build
- Check the output 
![alt text](<Images/Screenshot 2025-08-16 010343.png>)
![alt text](<Images/Screenshot 2025-08-16 010413.png>)
![alt text](<Images/Screenshot 2025-08-16 010431.png>)
### Successfully Deployed CICD pipeline
## Step 6) Access Node app
- Go to Node app server
- Copy the Public-IP of Node app server
- Hit on Browser with `Port-3000`
- open ` < Public-IP >:3000 ` in browser
![alt text](<Images/Screenshot 2025-08-16 010523.png>)
## Successfully deployed Node app CICD

### Check it on Node app server 
- Connect to server through SSH
- Check that the `node-app` directory is created or not
  ![alt text](<Images/Screenshot 2025-08-16 010918.png>)

### Jenkin server also created the Node app Directiory
- Connect to Jenkins server through SSH
- Go to workspace directory of jenkins, where jobs are created
```bash
cd /var/lib/jenkins/workspace
ls
cd node-app-CICD
```
![alt text](<Images/Screenshot 2025-08-16 011249.png>)

## Step 7) Add a Webhook
- Go to Github 
- Add a Webhook in GitHub repo
 ![alt text](<Images/Screenshot 2025-08-16 005550.png>)

- Go to `Settings → Webhooks → Add Webhook.`
 ![alt text](<Images/Screenshot 2025-08-16 011440.png>)
- In Webhook, click on `add webhook`
- In Payload URL : 
``` http://<JENKINS_PUBLIC_IP>:8080/github-webhook/ ```
![alt text](<Images/Screenshot 2025-08-16 011713.png>)
- Clicking on add, Webhook is been created
![alt text](<Images/Screenshot 2025-08-16 011732.png>)
 
### Make some changes in Github repo  
- Commit & push changes to GitHub.
- Webhook triggers Jenkins → Jenkins builds & deploys on EC2
- Access Node.js app: `<Node-app_PUBLIC_IP>:3000`
![alt text](<Images/Screenshot 2025-08-16 012028.png>)

- In Webhooks, it show the Trigger is Successfully delivered
![alt text](<Images/Screenshot 2025-08-16 012049.png>)

---
## Project Summary
This project demonstrates the end-to-end deployment of a Node.js application using a fully automated CI/CD pipeline with GitHub, Jenkins, and AWS EC2. The workflow integrates GitHub webhooks to trigger Jenkins pipelines on every code commit, installs dependencies, runs tests, and securely deploys the app to an EC2 instance using SSH credentials. With this setup, developers achieve continuous integration, continuous delivery, and automated deployment, ensuring faster releases, reduced manual effort, and a scalable cloud-based deployment environment.
