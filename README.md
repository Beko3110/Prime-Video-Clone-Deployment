# Amazon Prime Clone DevSecOps Project
The project outlined in the document involves deploying a clone of an Amazon Prime Video application using a comprehensive DevOps approach, primarily on AWS's EKS (Elastic Kubernetes Service) cluster. 

## Below is an overview of the stages, tools, and technologies involved in this project.

1. Containerization and Security Analysis
Objective: Containerize the application and enhance security.
Tools Used:
Docker: Used to containerize the application, ensuring it runs consistently across various environments.
SonarQube: Conducts code analysis, identifying vulnerabilities and ensuring code quality.
Trivy: Scans both the file system and container images to detect vulnerabilities.

2. CI/CD Pipeline
Objective: Establish a CI/CD pipeline for seamless deployment.
Tools Used:
Jenkins: Manages the entire CI/CD pipeline, from code checkout to notification upon success or failure.
SonarQube Integration: Integrated with Jenkins to analyze code during the pipeline process.
Docker Hub: Stores the container images post-build, allowing for easy retrieval and deployment.
Argo CD: Deployed to manage the EKS cluster, supporting GitOps for Kubernetes.

3. Deployment Stages
Steps:
Source Code Checkout: Initiated from a GitHub repository.
Code Analysis and Vulnerability Scanning: SonarQube and Trivy ensure the application is secure and free from vulnerabilities.
Container Image Build and Deployment: Jenkins builds and pushes the Docker image to Docker Hub, and the image is later deployed on the EKS cluster.

4. AWS Setup
Objective: Establish an environment on AWS for application deployment.
Tools Used:
AWS EKS: Manages the Kubernetes cluster for application deployment.
EC2 Instances: Various instances (like t2.large) host Jenkins, Docker, SonarQube, Trivy, and other services.
Security Groups: Configured to allow specific ports for application, SonarQube, Jenkins, and monitoring tools.

5. Monitoring and Logging
Objective: Ensure application and infrastructure performance visibility.
Tools Used:
Prometheus: Monitors the EKS cluster and its components.
Grafana: Visualizes metrics from Prometheus, providing a user-friendly dashboard for monitoring.

6. Final Steps and Notifications
Email Notifications: Configured via Jenkins for updates on pipeline execution status.
App Deployment Validation: Once the pipeline successfully builds and deploys the application, Jenkins sends the final success/failure notification.
This project integrates multiple DevOps and cloud tools, offering a robust end-to-end pipeline for deploying a high-availability application with secure, automated deployment and monitoring capabilities. Let me know if you’d like details on a specific step or tool.

### Process Flow Description:
#### Code Development & GitHub Repository:
Developers push code to the GitHub repository, triggering the pipeline in Jenkins.
#### CI/CD Pipeline in Jenkins:
#### Code Checkout: Jenkins retrieves the latest code from GitHub.
#### Quality Check: SonarQube scans the code for vulnerabilities and code quality issues.
#### Containerization: Docker builds an image of the application.
#### Security Scans: Trivy scans the Docker image for any vulnerabilities.
#### Docker Image Storage:
Jenkins pushes the Docker image to Docker Hub, where it’s stored for deployment.
#### Deployment with Argo CD and EKS:
Argo CD automates the deployment process, pulling the Docker image from Docker Hub and deploying it onto the EKS cluster.
AWS Security Groups ensure secure access and permissions for the EKS nodes and services.
#### Monitoring:
Prometheus collects metrics from the EKS cluster, nodes, and the application.
Grafana visualizes these metrics, enabling real-time monitoring of performance and application health.


## Getting Started with Create React App

### Available Scripts

In the project directory, you can run:

#### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

The page will reload when you make changes.\
You may also see any lint errors in the console.


##  Setup Guide
This guide describes the detailed steps to deploy an Amazon Prime clone application on AWS EKS using a DevSecOps approach, integrating multiple tools for CI/CD, security, and monitoring.

1. AWS CLI Installation
Install AWS CLI on your local Ubuntu machine to manage AWS services:
```bash

sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
2. Jenkins Installation on Ubuntu
Jenkins handles CI/CD, ensuring automated deployments and monitoring:
```bash

sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```
3. Docker Installation on Ubuntu
Docker enables containerization for deployment consistency across environments:
```bash

sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker ubuntu
sudo chmod 777 /var/run/docker.sock
newgrp docker
sudo systemctl status docker
```
4. Trivy Installation for Vulnerability Scanning
Install Trivy to scan images for vulnerabilities before deployment:
```bash

sudo apt-get install wget apt-transport-https gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```
5. Docker Scout Installation
Docker Scout enhances image security, scanning for vulnerabilities and providing insights:
```bash

docker login    # Provide DockerHub credentials
```

```bash
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin
```
6. Jenkins Pipeline Setup
In Jenkins, define stages in a pipeline to automate the CI/CD process:
```groovy

pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage ("clean workspace") {
            steps { cleanWs() }
        }
        stage ("Git checkout") {
            steps { git branch: 'main', url: 'https://github.com/Beko3110/Prime-Video-Clone-Deployment.git' }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=amazon-prime -Dsonar.projectKey=amazon-prime"
                }
            }
        }
        stage("quality gate") {
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            } 
        }
        stage("Install NPM Dependencies") {
            steps { sh "npm install" }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Trivy File Scan") {
            steps { sh "trivy fs . > trivy.txt" }
        }
        stage("Build Docker Image") {
            steps { sh "docker build -t amazon-prime ." }
        }
        stage("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag amazon-prime mohamed3110/amazon-prime:latest"
                        sh "docker push mohamed3110/amazon-prime:latest"
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                       sh 'docker-scout quickview mohamed3110/amazon-prime:latest'
                       sh 'docker-scout cves mohamed3110/amazon-prime:latest'
                       sh 'docker-scout recommendations mohamed3110/amazon-prime:latest'
                   }
                }
            }
        }
        stage("Deploy to Container") {
            steps {
                sh 'docker run -d --name amazon-prime -p 3000:3000 mohamed3110/amazon-prime:latest'
            }
        }
    }
    post {
        always {
            emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "<html>Pipeline Build Status</html>",
            to: 'mohamed.mustafa.soliman.31@gmail.com',
            mimeType: 'text/html',
            attachmentsPattern: 'trivy.txt'
        }
    }
}
```
7. Prometheus and Grafana for Monitoring
Install and configure Prometheus and Grafana to monitor the application and infrastructure health.
Prometheus Installation
```bash

sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
```

```bash
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

Set ownership for directories:
```bash
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```
Create a systemd unit configuration file for Prometheus:
```bash
sudo nano /etc/systemd/system/prometheus.service
```
Add the following content to the prometheus.service file:
```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```
Here's a brief explanation of the key parts in this prometheus.service file:

User and Group specify the Linux user and group under which Prometheus will run.

ExecStart is where you specify the Prometheus binary path, the location of the configuration file (prometheus.yml), the storage directory, and other settings.

web.listen-address configures Prometheus to listen on all network interfaces on port 9090.

web.enable-lifecycle allows for management of Prometheus through API calls.

Enable and start Prometheus:
```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```
Verify Prometheus's status:
```bash
sudo systemctl status prometheus
```
You can access Prometheus in a web browser using your server's IP and port 9090:
```bash
http://<your-server-ip>:9090
```
Installing Node Exporter:

Create a system user for Node Exporter and download Node Exporter:
```bash
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

Extract Node Exporter files, move the binary, and clean up:
```bash
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
```

Create a systemd unit configuration file for Node Exporter:
```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Add the following content to the node_exporter.service file:
```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
```

Replace --collector.logind with any additional flags as needed.

Enable and start Node Exporter:
```bash
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```
Verify the Node Exporter's status:
```bash
sudo systemctl status node_exporter
```

You can access Node Exporter metrics in Prometheus.

Configure Prometheus Plugin Integration:

Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

Prometheus Configuration:


Configure Prometheus with the node exporter and Jenkins:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<jenkins-ip>:<jenkins-port>']
```
- Make sure to replace <your-jenkins-ip> and <your-jenkins-port> with the appropriate values for your Jenkins setup.

Check the validity of the configuration file:
```bash
promtool check config /etc/prometheus/prometheus.yml
```
Reload the Prometheus configuration without restarting:
```bash
curl -X POST http://localhost:9090/-/reload
```
You can access Prometheus targets at:
```bash
http://<your-prometheus-ip>:9090/targets
```
Grafana Installation
Step 1: Install Dependencies:
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
```
Step 2: Add the GPG Key:

Add the GPG key for Grafana:
```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```
Step 3: Add Grafana Repository:

Add the repository for Grafana stable releases:
```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Step 4: Update and Install Grafana:

Update the package list and install Grafana:
```bash
sudo apt-get update
sudo apt-get -y install grafana
```

Step 5: Enable and Start Grafana Service:

To automatically start Grafana after a reboot, enable the service:
```bash
sudo systemctl enable grafana-server
```
Then, start Grafana:
```bash
sudo systemctl start grafana-server
```
Step 6: Check Grafana Status:
Verify the status of the Grafana service to ensure it's running correctly:
```bash
sudo systemctl status grafana-server
```
Step 7: Access Grafana Web Interface:

Open a web browser and navigate to Grafana using your server's IP address. The default port for Grafana is 3000. For example:
```bash
http://<your-server-ip>:3000
```

8. EKS Cluster Setup using eksctl
Create and configure an EKS cluster:
```bash
eksctl create cluster --name=amcdemo \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup 
```
# Get List of clusters
```bash
eksctl get cluster
```
Create & Associate IAM OIDC Provider for our EKS Cluster
To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create & associate OIDC identity provider.
To do so using eksctl we can use the below command.

```bash
# Replace with region & cluster name
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster amcdemo \
    --approve
```

```bash
# Create Public Node Group   
eksctl create nodegroup --cluster=amcdemo \
                       --region=us-east-1 \
                       --name=amcdemo-ng-public1 \
                       --node-type=t3.medium \
                       --nodes=2 \
                       --nodes-min=2 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=amc-demo \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```

9. Deploy Application with Argo CD
Install and configure Argo CD on your Kubernetes cluster to manage deployments.
After completing all steps, the Amazon Prime clone app will be securely deployed, monitored, and maintained in an automated DevSecOps environment on AWS EKS.

10. Argo CD Setup for Continuous Deployment
Argo CD is essential for GitOps in Kubernetes environments, automating application deployment and keeping clusters in sync with Git repositories.
Argo CD Installation
Create Namespace for Argo CD:
```bash

kubectl create namespace argocd
```

Install Argo CD:
```bash

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Configuring Argo CD
Access Argo CD UI:
Port-forward the Argo CD API server to access the UI:
```bash

kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Access the UI via https://localhost:8080. Log in using the default credentials (admin and the initial password from the Argo CD secret).
Set Up the GitHub Repository as a Source:
Connect Argo CD to your GitHub repository for the Prime Clone application.
Use Argo CD’s UI or CLI to configure the application’s sync settings and desired state.
Create and Configure Argo CD Application:
Define the application with the following specifications:
Name: Unique identifier for the app in Argo CD.
Project: Argo CD project the application belongs to.
Source: Git repository URL, branch, and path.
Destination: Kubernetes cluster and namespace where the application will be deployed.
Sync Policy: Set to automatically sync changes (enabling automatic deployments when code updates are detected).
Exposing the Application
Open Access to Application:
Ensure port 30007 (or desired port) is open in the security group for EKS.
Access the app at http://<node-IP>:30007.
11. Cluster and Node Verification
After configuring the cluster, verify everything is running as expected:
List EKS Clusters:
```bash

eksctl get cluster
```
Verify Node Groups and Nodes:
List node groups in the cluster:
```bash

eksctl get nodegroup --cluster=amcdemo
```
Check the nodes and their status:
```bash

kubectl get nodes -o wide
```
Confirm kubectl Context:
Ensure kubectl context is set to the new cluster by running:
```bash

kubectl config view --minify
```
12. Application and CI/CD Monitoring with Prometheus and Grafana
Prometheus and Grafana provide real-time monitoring of application performance, infrastructure, and CI/CD processes.
Configure Prometheus Targets:
Add Jenkins and Node Exporter targets in prometheus.yml for scraping:
```yaml

scrape_configs:
  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<jenkins-ip>:<jenkins-port>']
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```
Verify the configuration:
```bash

promtool check config /etc/prometheus/prometheus.yml
```
Reload Prometheus configuration:
```bash

curl -X POST http://localhost:9090/-/reload
```
Set Up Grafana Dashboards:
Access Grafana (http://<server-ip>:3000) and log in.
Add Prometheus as a Data Source:
Navigate to Data Sources > Add Data Source > Prometheus, then enter the Prometheus URL (http://<prometheus-server-ip>:9090).
Import Dashboards for Jenkins and Node Exporter:
Add pre-configured Grafana dashboards using IDs (e.g., Jenkins dashboard ID 9964, Node Exporter dashboard ID 1860).
13. Cleanup
After deployment and testing, you can clean up to prevent unwanted costs:
Delete EKS Node Group and Cluster:
Delete Node Group:
```bash

eksctl delete nodegroup --cluster=amcdemo --name=amcdemo-ng-public
```
Delete Cluster:
```bash

eksctl delete cluster amcdemo
```
Terminate EC2 Instances:
Identify and terminate EC2 instances created for Jenkins, Prometheus, Grafana, and other components.
Remove Additional AWS Resources:
Manually check for any additional resources (e.g., IAM roles, security groups) and remove those specific to the project.

## Summary of DevSecOps Workflow
This setup represents a DevSecOps approach, emphasizing continuous integration and deployment with integrated security and monitoring:
Continuous Integration and Testing: Jenkins CI/CD with SonarQube and Trivy ensures code quality and security checks at each stage.
Automated Deployment: Argo CD applies a GitOps strategy, auto-syncing changes to the EKS cluster.
Vulnerability Scanning and Reporting: Docker Scout and OWASP Dependency Check enhance the security of Docker images.
Real-Time Monitoring: Prometheus and Grafana dashboards offer insights into the application and infrastructure.
This comprehensive DevSecOps architecture enables secure, automated, and observable deployment of the Amazon Prime clone on AWS, making it ideal for production-level readiness. Let me know if further assistance is needed on any part!

