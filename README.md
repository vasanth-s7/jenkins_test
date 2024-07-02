# To-Do List CI/CD using jenkins, github web hooks, Dockerhub & Docker
### Dockerhub project link:- https://hub.docker.com/repository/docker/ravalikadocker/to-do/general
### image file :- docker pull ravalikadocker/to-do:latest

## Prerequisites
1. **Jenkins**: Ensure Jenkins is installed and running.
2. **Docker**: Install Docker on the Jenkins server and agents.
3. **Docker Compose**: Install Docker Compose on the Jenkins server and agents.
4. **GitHub Repository**: The repository containing the To-Do List application.
5. **Docker Hub Credentials**: Store your Docker Hub credentials in Jenkins with the ID 'dc'.

# Automated Installation of Docker, Jenkins, and Docker Compose

This repository contains a shell script to automate the installation of Docker, Docker Compose, and Jenkins on an Ubuntu 22.04 system.

## Prerequisites

- Ubuntu 22.04 system
- Sudo privileges

## Installation

1. Clone this repository to your local machine:

    ```sh
    git clone https://github.com/Ravalika-j/jenkins-cicd.git
    ```

2. Make the script executable:

    ```sh
    chmod +x install_cicd.sh
    ```

3. Run the script:

    ```sh
    ./install_cicd.sh
    ```

## Script Details

The `install_cicd.sh` script performs the following steps:

1. Updates the package list and installs Docker and Docker Compose.
2. Adds the current user to the `docker` group.
3. Installs OpenJDK 17, required for Jenkins.
4. Adds the Jenkins repository and key.
5. Installs Jenkins.
6. Starts the Jenkins service and waits for it to initialize.
7. Displays the initial Jenkins admin password.
8. Adds the Jenkins user to the `docker` group.
9. Reboots the system to apply group changes.

## Post-Installation

After the system reboots, Jenkins will be running. Access Jenkins at:

    http://localhost:8080
for AES EC2 instance make sure you open 8080 port from security group and also replace localhost with EC2 servers IP address

Follow the Jenkins setup wizard to complete the installation.

## Notes

- The script includes a reboot command to apply group changes. Ensure you save any work before running the script.
- You might need to log out and log back in for the group changes to take effect for your user.

---

# Manual installation guide : (skip this if you follow my automated installation using shell script step)

## Installation Steps

### 1. Docker and Docker Compose Installation in your prefered (system aws EC2)

```bash
$ sudo apt-get update
$ sudo apt install docker.io -y
$ sudo apt install docker-compose -y
$ sudo usermod -aG docker $USER
$ sudo reboot

# After reboot
$ docker ps
```
### 2. Jenkins Installation

```bash
$ sudo apt install openjdk-17-jre-headless -y
$ java --version
$ sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
$ echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
$ sudo apt-get update
$ sudo apt-get install jenkins -y 
$ systemctl status jenkins
$ sudo usermod -aG docker jenkins
$ sudo reboot

# After reboot
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
### 3. Git Installation (if not already installed)

```bash
$ git --version
$ sudo apt install git -y
```
### 4. Initial Docker Setup for the Application
```bash
$ docker build -t to-do .
$ docker run -itd -p 80:80 --name to-do to-do
$ docker stop to-do
$ docker rm to-do
```
# (optional step ends here)

## Setting Up the Pipeline

1. **Clone the repository**: Ensure the repository is accessible from Jenkins.
2. **Configure Jenkins**: 
    - Install the necessary Jenkins plugins: Git, Docker Pipeline, Credentials Binding.
    - Create a Jenkins job and configure the pipeline script.

3. **Add Credentials**:
    - Go to `Jenkins -> Manage Jenkins -> Manage Credentials`.
    - Add a new entry for Docker Hub with the ID 'dc'.

4. **Run the Pipeline**: Trigger the Jenkins job to run the pipeline.

## Enabling GitHub Webhooks

To automate the CI/CD pipeline, you can configure GitHub webhooks to trigger the Jenkins pipeline on code changes.

1. **Setup Webhook in GitHub**:
    - Navigate to your repository on GitHub.
    - Go to `Settings -> Webhooks -> Add webhook`.
    - In the `Payload URL` field, enter your Jenkins URL followed by `/github-webhook/`. For example, `http://your-jenkins-url/github-webhook/`.
    - Set the `Content type` to `application/json`.
    - Select `Just the push event` or choose specific events according to your needs.
    - Click `Add webhook`.

2. **Configure Jenkins Job**:
    - Open your Jenkins job configuration.
    - In the `Build Triggers` section, check the `GitHub hook trigger for GITScm polling` option.

3. **Ensure GitHub Plugin is Installed**:
    - Go to `Manage Jenkins -> Manage Plugins`.
    - Ensure the `GitHub Integration Plugin` is installed.

Once the webhook is set up, any push to the GitHub repository will trigger the Jenkins pipeline, automating the build, push, deploy, and test processes.

## Pipeline Script

This CI/CD pipeline automates the process of building, pushing, and deploying the To-Do List application. It ensures that the application is always up-to-date and tested, providing a reliable and efficient workflow.

For any issues or contributions, please raise an issue or submit a pull request in the repository or email at ravalikajalla123@gmail.com.

The pipeline performs the following steps:
1. Clones the repository from GitHub.
2. Builds the Docker image for the application.
3. Pushes the Docker image to Docker Hub.
4. Deploys the application using Docker Compose.
5. Runs tests to ensure the application is working correctly.

Here's the complete Jenkins pipeline script:

```groovy
pipeline {
    agent any

    stages {
        stage("Code") {
            steps {
                echo "Cloning the code"
                git url: "https://github.com/Ravalika-j/jenkins-cicd.git", branch: "main"
            }
        }
        stage("Test") {
            steps {
                echo "Running tests"
                // Add your test commands here
                // e.g., sh "pytest tests/"
            }
        }
        stage("Build") {
            steps {
                echo "Building the code"
                sh "docker build -t to-do ."
            }
        }
        stage("Push to dockerhub") {
            steps {
                echo "Pushing the code to Docker Hub"
                withCredentials([usernamePassword(credentialsId: 'dc', passwordVariable: 'dockerhubpass', usernameVariable: 'dockerhubusr')]) {
                    sh """
                        docker login -u ${dockerhubusr} -p ${dockerhubpass}
                        docker tag to-do ${dockerhubusr}/to-do:latest
                        docker push ${dockerhubusr}/to-do:latest
                    """
                }
            }
        }
        stage("Deploy") {
            steps {
                echo "Deploying the code to agents"
                sh """
                    docker-compose down
                    docker-compose up -d
                """
            }
        }
    }
}

}
```
Conclusion
This CI/CD pipeline automates the process of building, pushing, and deploying the To-Do List application. It ensures that the application is always up-to-date and tested, providing a reliable and efficient workflow.

For any issues or contributions, please raise an issue or submit a pull request in the repository or email at ravalikajalla123@gmail.com.

 
