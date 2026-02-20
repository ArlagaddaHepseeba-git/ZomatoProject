 # Containerizing the Zomato App using Docker by implementing CI/CD tool Jenkins 🚀
 ## Project Overview
In this project, we use Jenkins to build a CI/CD pipeline. This pipeline connects tools like Docker, Trivy, SonarQube, and OWASP Dependency Check all together. These tools help us check the code quality and find  any security problems before we deploy the application.
## Project Architecture
<img width="2240" height="1260" alt="image" src="https://github.com/user-attachments/assets/e011e96c-570a-4dde-969f-080393016058" />
## Tools and Technologies Used
| 🛠 Technology | Purpose |
|--------------|---------|
| **AWS EC2 (t2.large)** | Hosts Jenkins, SonarQube, and the application on the cloud |
| **Amazon Linux 2023** | Operating system running on the EC2 instance |
| **Jenkins (v2.541.2)** | Automates the entire CI/CD pipeline from build to deployment |
| **Docker (v25.0.14)** | Builds and runs the application inside containers |
| **SonarQube (v9.9.8)** | Performs code quality analysis and enforces quality gates |
| **OWASP Dependency Check** | Scans npm packages for known security vulnerabilities |
| **Trivy (v0.18.3)** | Scans filesystem and Docker images for security issues |
| **GitHub** | Stores and manages the source code |
| **DockerHub** | Stores the built Docker images for easy access |
| **React & Node.js (v16)** | Frontend framework used to build the Zomato application |
| **Nginx (Alpine)** | Lightweight web server that serves the React application |



