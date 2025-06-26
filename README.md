# CI/CD Pipeline with Terraform, Jenkins, Docker & SonarQube

This project sets up a CI/CD pipeline using:
- **Terraform** to provision an AWS EC2 instance (t3.large)
- **Jenkins** for build automation
- **SonarQube** for static code analysis
- **Docker** for containerization
- **Trivy** for security scanning

---

## 🚀 Step 1: Provision EC2 Using Terraform

```hcl
resource "aws_instance" "jenkins_server" {
  ami           = "ami-xxxxxxx"        # Use Ubuntu AMI ID
  instance_type = "t3.large"
  key_name      = "your-key"
  tags = {
    Name = "Jenkins-Server"
  }
}


🔧 Step 2: Install Jenkins and Fetch Admin Password
SSH into your EC2 instance and run:

sudo apt update && sudo apt install -y openjdk-17-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Fetch Jenkins Admin Password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword


🐳 Step 3: Install SonarQube via Docker
docker run -d --name sonarqube -p 9000:9000 sonarqube:community




🧩 Step 4: Jenkins Plugin Installation
Navigate to Manage Jenkins → Plugins → Available and install the following plugins:

Eclipse Temurin Installer

SonarQube Scanner

Sonar Quality Gates

Quality Gates

NodeJS

All Docker-related plugins

Stage View

Pipeline



🛠️ Step 5: Jenkins Tool Configuration
Go to Manage Jenkins → Global Tool Configuration and configure:

✅ Node.js:
Name: node16

Install automatically

Version: 21.20.0 from nodejs.org

✅ JDK:
Name: jdk17

Install from adoptium.net

Version: jdk-17.0.8.1+1

✅ Docker:
Name: docker

Install automatically from docker.com

✅ SonarQube Scanner:
Name: sonarqube-scanner

Install automatically from Maven Central (latest version)

⚙️ Step 6: SonarQube Setup & Integration with Jenkins
🔐 Generate Token in SonarQube
Go to http://<sonarqube-ip>:9000

Login with:

Username: admin

Password: admin (change it)

Go to Administration → Security → Tokens → Generate Token

🧾 Add Token to Jenkins
Go to Manage Jenkins → Credentials → Global → Add Credentials

Kind: Secret text

ID: SonarQube-Token

Secret: Paste the token

🖇️ Configure SonarQube Server in Jenkins
Go to Manage Jenkins → Configure System

Scroll to SonarQube Servers

Name: SonarQube-Server

Server URL: http://<sonarqube-ip>:9000

Token: Use SonarQube-Token from credentials


📊 Step 7: Configure SonarQube Quality Gate & Webhook
➕ Quality Gate:
Go to SonarQube Dashboard → Quality Gates → Create

Name: SonarQube-Quality-Gate

🔔 Webhook:
Go to Administration → Configuration → Webhooks

Name: jenkins

URL: http://<jenkins-ip>:8080/sonarqube-webhook/

🔄 Step 8: Jenkins Pipeline - CI/CD
✅ Create New Item:
Name: swiggy-cicd

Type: Pipeline

📜 Pipeline Script:

pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Ashfaque-9x/a-swiggy-clone.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy-CI \
                    -Dsonar.projectKey=Swiggy-CI'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh "docker build -t swiggy-clone ."
                        sh "docker tag swiggy-clone ashfaque9x/swiggy-clone:latest"
                        sh "docker push ashfaque9x/swiggy-clone:latest"
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image ashfaque9x/swiggy-clone:latest > trivyimage.txt"
            }
        }
    }
}

🐳 Step 9: Push to DockerHub
🧾 DockerHub Token:
Go to DockerHub → Settings → Security → Generate Access Token

🧾 Add Credential in Jenkins:
Go to Manage Jenkins → Credentials → Global → Add Credentials

Kind: Username with password

Username: DockerHub username

Password: Paste access token

ID: dockerhub

✅ Final Steps
Click on Build Now in Jenkins

Monitor pipeline stages

Visit SonarQube dashboard to verify the analysis

Verify Docker image is pushed to DockerHub



