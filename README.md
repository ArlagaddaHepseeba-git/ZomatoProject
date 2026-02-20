 # Containerizing the Zomato App using Docker by implementing CI/CD tool Jenkins 🚀
 ## Project Overview
In this project, we use Jenkins to build a CI/CD pipeline. This pipeline connects tools like Docker, Trivy, SonarQube, and OWASP Dependency Check all together. These tools help us check the code quality and find  any security problems before we deploy the application.
## Project Architecture
<img width="2240" height="1260" alt="image" src="https://github.com/user-attachments/assets/e011e96c-570a-4dde-969f-080393016058" />
## Tools and Technologies Used
Tool  	 Purpose
AWS EC2	Cloud server to host everything
GitHub 	Store the source code
Jenkins 	CI/CD automation tool
SonarQube	 Check code quality
Node.js 	 Run and build the React application
OWASPDependencyCheck 	Scan for vulnerable libraries
Docker 	 Containerize and deploy the app
Trivy 	 Scan files and Docker images for security issues
DockerHub 	Store the Docker image 

### Step 1: Launch AWS EC2 Instance
I launched an EC2 instance on AWS to host the entire DevOps toolchain and application.
Instance Details:
AMI: Amazon Linux 2023
Instance Type: t2.large (2 vCPU, 8 GB RAM)
Storage: 28 GB gp3
Key Pair: myapp-key.pem 
<img width="1920" height="1080" alt="EC2-Instance" src="https://github.com/user-attachments/assets/e3de2cb4-7a7d-4d85-9081-aecdf9fcf86f" />
### Step 2: Install Jenkins
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install java-17-amazon-corretto -y
yum install jenkins -y
systemctl start jenkins.service
systemctl enable jenkins.service
systemctl status jenkins.service
### Step 3: Install Docker and Git
yum install docker git -y
systemctl start docker
systemctl status docker
chmod 777 /var/run/docker.sock
### Step 4: Install SonarQube (using Docker)
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
Access SonarQube at: `http://<your-ec2-public-ip>:9000`
-> Default username: `admin`
-> Default password: `admin`
-> Update password on first login
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
### Step 7: Configure Tools in Jenkins
Go to **Manage Jenkins → Tools** and configure:
**JDK** → Name: `jdk17`, Install from adoptium.net
**NodeJS** → Name: `node16`, Version: NodeJS 16.2.0
**SonarQube Scanner** → Name: `mysonar`, Install from Maven Central
**Dependency-Check** → Name: `DP-Check`, Install automatically
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
### Step 10: Create Jenkins Pipeline Job

Go to **Jenkins Dashboard → New Item**

- Name: `myDeployment`
- Type: **Pipeline**
- Click OK

Paste the following pipeline script:
```groovy
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
## Dockerfile
```dockerfile
FROM node:16 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

This Dockerfile uses a **multi-stage build**:
- First stage builds the React app using Node.js
- Second stage serves the built files using a lightweight Nginx web server



