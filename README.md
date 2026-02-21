 # Containerizing the Zomato App using Docker by implementing CI/CD tool Jenkins 🚀
 ## Project Overview
This project uses Jenkins to create an automated pipeline. It connects tools like Docker, Trivy, SonarQube, and OWASP to check for security issues and code quality. This helps us deliver safe and reliable software
## Project Architecture
<img width="2240" height="1260" alt="image" src="https://github.com/user-attachments/assets/e011e96c-570a-4dde-969f-080393016058" />

## Tools and Technologies Used

**Tool**  	              **Purpose**

AWS EC2	               Cloud server to host everything

GitHub 	               Store the source code

Jenkins 	              CI/CD automation tool

SonarQube	             Check code quality

Node.js 	             Run and build the React application

OWASPDependencyCheck 	Scan for vulnerable libraries

Docker 	              Containerize and deploy the app

Trivy 	               Scan files and Docker images for security issues

DockerHub 	           Store the Docker image 

### Step 1: Launch AWS EC2 Instance

I launched an EC2 instance on AWS to host the entire DevOps toolchain and application.
## Instance Details:
# AMI: Amazon Linux 2023

# Instance Type: t2.large (2 vCPU, 8 GB RAM)

# Storage: 28 GB gp3

# Key Pair: myapp-key.pem 

<img width="1920" height="1080" alt="EC2-Instance" src="https://github.com/user-attachments/assets/e3de2cb4-7a7d-4d85-9081-aecdf9fcf86f" />

### Step 2: Install Jenkins
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install java-17-amazon-corretto -y
yum install jenkins -y
systemctl start jenkins.service
systemctl enable jenkins.service
systemctl status jenkins.service
<img width="1920" height="1080" alt="instal all" src="https://github.com/user-attachments/assets/91ea4be1-8fa2-4810-bfa5-e902c723c7e5" />
### Step 3: Install Docker and Git
yum install docker git -y
systemctl start docker
systemctl status docker
chmod 777 /var/run/docker.sock
### Step 4: Install SonarQube (using Docker)
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
Access SonarQube at: `http://<your-ec2-public-ip>:9000`
-> Default username: admin
-> Default password: admin
-> Update password on first login
<img width="1920" height="1080" alt="sonarr login" src="https://github.com/user-attachments/assets/164d8cc5-c452-417e-8dca-114947b6d6b0" />
<img width="1920" height="1080" alt="update sonar" src="https://github.com/user-attachments/assets/d569f2f5-42a7-4977-a17d-df6911e0c286" />
### Step 5: Install Trivy
wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.tar.gz
tar zxvf trivy_0.18.3_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/
echo 'export PATH=$PATH:/usr/local/bin/' >> ~/.bashrc
source ~/.bashrc
trivy --version
### Step 6: Install Jenkins Plugins
Go to **Jenkins Dashboard → Manage Jenkins → Plugins → Available Plugins**
Install the following plugins:
-> SonarQube Scanner
->Eclipse Temurin Installer
->NodeJS
->OWASP Dependency-Check
->Docker Pipeline
->pipeline stage view
<img width="1920" height="1080" alt="all plugins installed" src="https://github.com/user-attachments/assets/24d8dccf-7259-4df8-880b-d659495e1a80" />
### Step 7: Configure Tools in Jenkins
Go to **Manage Jenkins → Tools** and configure:
**JDK** → Name: `jdk17`, Install from adoptium.net
<img width="1920" height="1080" alt="jdk17" src="https://github.com/user-attachments/assets/13c38cd2-cb32-47dc-8a5c-e91915cdb785" />
**NodeJS** → Name: `node16`, Version: NodeJS 16.2.0
<img width="1920" height="1080" alt="node16" src="https://github.com/user-attachments/assets/20009580-5096-4dbe-9744-992366fbd77b" />
**SonarQube Scanner** → Name: `mysonar`, Install from Maven Central
<img width="1920" height="1080" alt="mysonar" src="https://github.com/user-attachments/assets/ccccd7c6-4974-4cb4-bb8a-be3d8b4e8619" />
**Dependency-Check** → Name: `DP-Check`, Install automatically
<img width="1920" height="1080" alt="dp-check" src="https://github.com/user-attachments/assets/f42a13aa-bf1f-4f3d-a8e2-fbc465ee0579" />
### Step 8: Connect SonarQube with Jenkins
1. In SonarQube → **Administration → Security → Users → Tokens**
2. Generate a new token and copy it
3. In Jenkins → **Manage Jenkins → Credentials** → Add new **Secret Text**
   - Secret: your SonarQube token
   - ID: `sonar-token`
4. In Jenkins → **Manage Jenkins → System** → Add SonarQube server
   - Name: `mysonar`
   - URL: `http://<your-ec2-ip>:9000`
   - Token: select `sonar-token`
 ### Step 9: Add DockerHub Credentials in Jenkins
Go to **Manage Jenkins → Credentials → Global → Add Credentials**
- Kind: Username and Password
- Username: your DockerHub username
- Password: your DockerHub password
- ID: `dockerhub`
- 
### Step 10: Create Jenkins Pipeline Job

Go to **Jenkins Dashboard → New Item**

- Name: `myDeployment`
- Type: **Pipeline**
- 
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'mysonar'
    }
    stages {

        stage("clean workspace") {
            steps {
                cleanWs()
            }
        }

        stage("code") {
            steps {
                git branch: 'main', url: 'https://github.com/ArlagaddaHepseeba-git/ZomatoProject.git'
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }

        stage("Install Dependencies") {
            steps {
                sh 'npm install'
            }
        }

        stage("OWASP dependency") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --disableOssIndex', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Dockerfile") {
            steps {
                sh 'docker build -t hepseeba/hepdockerrepo:zomato .'
            }
        }

        stage("Trivy") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage("Trivy Image Scan") {
            steps {
                sh 'trivy image hepseeba/hepdockerrepo:zomato'
            }
        }

        stage("DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                        sh 'docker push hepseeba/hepdockerrepo:zomato'
                    }
                }
            }
        }

        stage("Deploy Container") {
            steps {
                sh 'docker stop zomato || true'
                sh 'docker rm zomato || true'
                sh 'docker run -d --name zomato -p 3000:80 hepseeba/hepdockerrepo:zomato'
            }
        }

    }
}


  
===============
<img width="1920" height="1080" alt="pipeline build" src="https://github.com/user-attachments/assets/665e67aa-ba42-4016-8fd1-74f2a27e6d49" />

<img width="1920" height="1080" alt="Screenshot 2026-02-19 174311" src="https://github.com/user-attachments/assets/efe7e84b-7d36-4506-983f-4558b3dd35c9" />
<img width="1400" height="584" alt="image" src="https://github.com/user-attachments/assets/f88bfbf3-0082-412e-b2c9-0f5dad5d6016" />


##Docker hub registry image
<img width="1920" height="1080" alt="hub" src="https://github.com/user-attachments/assets/4a7918e4-49b9-4d49-ae25-b9e1ccf182fd" />
<img width="1920" height="1080" alt="dockerhub" src="https://github.com/user-attachments/assets/52b05eae-fe17-4edd-be3f-b8bfcdb27baf" />
<img width="1920" height="1080" alt="zomato app" src="https://github.com/user-attachments/assets/75f911c1-c7ea-4302-ae55-9f02e473d797" />








