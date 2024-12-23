# Jenkins Setup and Pipeline Creation

## Steps to Set Up Jenkins and Create a Basic Pipeline

### Prerequisites :
- Docker Installation
- Ubuntu OS
- Jenkins Installation
- AWS EC2 Instance
- OpenJDK JAVA Installation

### 1. Update and Upgrade

Run the following commands to ensure your system is up-to-date:
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Docker

Install Docker using the following commands:
```bash
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```
Log out and log back in to apply group changes.

### 3. Install OpenJDK 11

Install Java Development Kit version 11:
```bash
sudo apt install -y openjdk-11-jdk
```
Verify the installation:
```bash
java -version
```

### 4. Install Jenkins

Run the Jenkins container using Docker:
```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```
Access Jenkins by navigating to `http://<your-instance-ip>:8080` in your browser.
Retrieve the initial admin password from the Jenkins container:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
Follow the on-screen instructions to complete the setup.

### 5. Add the AWS EC2 Instance as a Node

Install Java on the AWS EC2 instance (required for the Jenkins agent):
```bash
sudo apt update && sudo apt install -y openjdk-11-jdk
```
In Jenkins, navigate to **Manage Jenkins > Manage Nodes and Clouds > New Node**.
Provide a name for the node, select **Permanent Agent**, and configure the following:
- **Remote root directory**: A directory on the EC2 instance, e.g., `/home/jenkins`.
- **Launch method**: Select "Launch agent via SSH."
- **Host**: Provide the public IP or hostname of the EC2 instance.
- **Credentials**: Add SSH credentials (private key or username/password) for the EC2 instance.

Save and connect the node. Jenkins will establish an SSH connection to the EC2 instance and set it up as an agent.

### 6. Write a Basic Pipeline to Install Nginx on the Node

Create a new pipeline job in Jenkins.
Add the following script to the pipeline definition:
```groovy
pipeline {
    agent { label 'jenkins-nginx' }
    stages {
        stage('Setup') {
            steps {
                sh '''
                sudo apt update && sudo apt upgrade -y
                sudo apt install -y docker.io
                sudo systemctl start docker
                sudo systemctl enable docker
                '''
            }
        }
        stage('Install Nginx') {
            steps {
                script {
                    // Check if the container exists
                    def containerExists = sh(script: "sudo docker ps -a -q -f name=nginx", returnStdout: true).trim()
                    
                    if (containerExists) {
                        echo "Stopping and removing existing 'nginx' container..."
                        sh '''
                        sudo docker stop nginx
                        sudo docker rm nginx
                        '''
                    } else {
                        echo "Container 'nginx' does not exist. Proceeding to run a new container."
                    }

                    // Start a new Nginx container
                    sh '''
                    sudo docker run --name nginx -d -p 80:80 nginx
                    '''
                }
            }
        }
    }
}

```
Save and run the pipeline. This will install and run an Nginx container on the node.

