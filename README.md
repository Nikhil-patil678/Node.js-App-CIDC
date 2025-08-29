# Node.js App Deployment using jenkins pipeline

> This project helps you how to deploy a Node.js application on a **Remote server** using jenkins pipeline with PM2 as a process manager .

## Objective :

* #### Automate build , test , and deployment of a node.js app .

* #### Run the app on Remote Server instead of the jenkins host .

* #### Deploy and manage the app with PM2 .

## Teck Stack :

- **node.js** : Application runtime

- **npm**     :  Package manager

- **jenkins** : CI/CD pipeline automation 

- **pm2**     : Process manager 

- **Github**  : Source code management 

- **Ubuntu**  : Remote deployment server 

## Prerequisites :
1. ### Infrastructure 

   * **Jenkins Server** (to run the pipeline)

   * **Remote Server** (to host the application)

2. ### Software Requirements 

   * Remote Server with :
     * Node.js & NPM installed 

     * PM2 installed 

   * Jenkins server with :
     * Jenkins installed 

     * Java installed 

     * Plugins installed 

     * Github repository with your node.js application 

   * Plugins Used : 
      * [Github plugin](https://plugins.jenkins.io/github/)
   
     * [SSh Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

     * [Pipeline Plugin](https://plugins.jenkins.io/workflow-aggregator/)

3. ### Security Groups 

    * Allow Port :
      * 22 ( For SSH Access)
      
      * 8080 (For Jenkins)

      * 3000 ( For Node.js)
   
* Both server launch using the same key 

## Environment Steup :

* ### Setup Jenkins Server 
 
  * Java installation :

  bash
 
      sudo apt update 
 
      sudo apt install openjdk-17-jdk -y

  
* ### Setup Remote Server 
    
    * Node.js installation :

  bash 
        
      sudo apt update 

      sudo apt install -y node.js  

   * NPM installation :

  bash

      sudo apt install npm

   * PM2 installation :

  bash 

      sudo npm install -g pm2 

#### Jenkins Setup 
![my image](./Images/Screenshot%202025-08-29%20055840.png)

#### Remote Setup 
![my image](./Images/Screenshot%202025-08-29%20055255.png)


## Setting Up Jenkins Credentials 

### Step 1 : Copy your pem key on jenkins server 

#### From your local machine : 
bash 

      scp -i pem-linux-key.pem pem-linux-key.pem ubuntu@<jenkins-server-public-ip>:/home/ubuntu


### Step 2 : Add Pem Key To Jenkins Credentials 

1. Go to settings --> credentials --> Under stored scoped to jenkins -> Click (global) --> Add credentials 

2. #### Fill In : 
   *  **Kind** : SSH Username with private key 
   
   *  **ID**   : `node-appkey-credentials` 
   
   *  **Username** : `Ubuntu`
   
   *  **private key** : Choose **Enter directly** and paste the contents of `pem-key-server.pem`

![my image](./Images/Screenshot%202025-08-29%20065121.png)

3. #### Click Create :

![my image](./Images/Screenshot%202025-08-29%20070253.png)


## Create jenkins Pipeline Job

1. **Create Job** : Node-app-deployment 

2. **Job type**  : Pipeline 

3. Under **Pipeline script from scm** : 
      * Choose : Git 

      * Connect to your github repo 

![my image](./Images/Screenshot%202025-08-29%20072405.png)

4. Add a jenkinsfile in your repo 
---
### Jenkinsfile for Remote Deployment

  bash


    pipeline {
    agent any

    environment {
        SERVER_IP      = <public-ip-of-app-server>
        SSH_CREDENTIAL = 'node-app-key-credentials'
        REPO_URL       = 'https://github.com/Nikhil-patil678/Node.js-App-CIDC.git'
        BRANCH         = 'main'
        REMOTE_USER    = 'ubuntu'
        REMOTE_PATH    = '/home/ubuntu/node-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Upload Files to EC2') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'mkdir -p ${REMOTE_PATH}'
                        scp -o StrictHostKeyChecking=no -r * ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}/
                    """
                }
            }
        }

        stage('Install Dependencies & Start App') {
            steps {
                sshagent([SSH_CREDENTIAL]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            cd ${REMOTE_PATH} &&
                            npm install &&
                            pm2 start app.js --name node-app || pm2 restart node-app
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Application deployed successfully!'
        }
        failure {
            echo '❌ Deployment failed.'
        }
      }
    }

---

5. Build the job :

#### Console Output Showing Succeesful Deployment 

![my image](./Images/Screenshot%202025-08-29%20175338.png)

* ### Access URL : 

bash 

    http://<public-ip of app-server>:3000


## Setup Github Hooks 

- Go to github repo -->  **settings** -> **Webhooks** --> **Add Webhooks** --> 

- Payload URL :  http://< jenkins-server-ip >:8080/github-webhook/

- Content Type : application/x-www-form-urlencoded 

- Trigger : Just the push event


## Summary : 

#### Automated deployment of a Node.js application on a remote Ubuntu server using Jenkins CI/CD pipeline with GitHub integration and PM2 process management.

## Author : 

**NIKHIL PATIL**

📧 np3250072@gmail.com

🔗 **[Linkdein]** : https://www.linkedin.com/in/nikhil-patil-259132306/ 

🔗 **[Github]**  : https://github.com/Nikhil-patil678 

---









         