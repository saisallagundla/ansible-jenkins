Project Overview: CI/CD Pipeline with Jenkins, Ansible, Docker, and GitHub on AWS
This project outlines the process of setting up a CI/CD (Continuous Integration/Continuous Deployment) pipeline using Jenkins, Ansible, Docker, and GitHub on AWS. The pipeline is designed to automatically build, test, and deploy a Dockerized web application to an AWS EC2 instance whenever changes are committed to a GitHub repository. This detailed guide walks through each step, explaining the key commands and configurations used.

Step 1: Push Code to GitHub Repository
Before setting up Jenkins or other infrastructure, the first step is to store your code in a GitHub repository. This allows Jenkins to pull the latest code automatically whenever changes are committed.

Create a GitHub Repository:

Visit GitHub and create a new repository named ansible-jenkins.
Once the repository is created, clone it to your local machine using the following command:
bash
Copy code
git clone https://github.com/yourusername/ansible-jenkins.git
cd ansible-jenkins
This command creates a local copy of your repository where you can manage your project files.

Add Project Files and Push to GitHub:

Initialize the repository and add your project files:
bash
Copy code
git init
git add .
Commit your changes with a meaningful message:
bash
Copy code
git commit -m "Initial commit of the project"
Link your local repository to the GitHub repository and push the changes:
bash
Copy code
git remote add origin https://github.com/yourusername/ansible-jenkins.git
git push -u origin main
Explanation:

git init initializes a new Git repository in your project folder.
git add . stages all the files in the current directory for commit.
git commit -m "message" commits the staged files with a descriptive message.
git remote add origin <URL> connects the local repository to the remote repository hosted on GitHub.
git push -u origin main pushes the committed files to the main branch on GitHub.
This step ensures that your codebase is version-controlled and accessible to Jenkins.

Step 2: Set Up GitHub Webhook to Trigger Jenkins
To automate the deployment process, set up a webhook in GitHub to trigger Jenkins whenever code is pushed to the repository.

Configure GitHub Webhook:

In your GitHub repository, navigate to Settings > Webhooks.

Click on Add webhook.

Set the Payload URL to your Jenkins server URL followed by /github-webhook/. For example:

arduino
Copy code
http://<jenkins-instance-ip>:8080/github-webhook/
Set the Content type to application/json.

Choose Just the push event to trigger the build on every push to the repository.

Click Add webhook.

Explanation:

A webhook is a way for an application (GitHub) to send real-time data to another application (Jenkins). In this case, the webhook notifies Jenkins whenever a push event occurs in the GitHub repository, triggering the Jenkins pipeline.
Step 3: Provision AWS EC2 Instances
Provisioning EC2 instances on AWS is necessary to host Jenkins and Docker, which will be used to manage and deploy the application.

Create Two Ubuntu EC2 Instances:

Jenkins Instance: This instance will host Jenkins, which will manage the CI/CD pipeline.
Docker Instance: This instance will run Docker containers that host the application.
Steps:

Log in to the AWS Management Console.
Navigate to EC2 > Instances and click on Launch Instance.
Choose an Ubuntu AMI (Amazon Machine Image).
Select an appropriate instance type (e.g., t2.micro for free tier).
Configure instance details and launch the instance with an existing or new key pair.
Configure Security Groups:

Jenkins Instance:
Open port 8080 (HTTP) for accessing Jenkins.
Open port 22 (SSH) for secure shell access.
Docker Instance:
Open port 80 (HTTP) for the web server.
Open port 443 (HTTPS) for secure web traffic.
Open port 22 (SSH) for remote access.
Explanation:

Security groups act as virtual firewalls that control inbound and outbound traffic to your EC2 instances. Opening the correct ports ensures that the services running on the instances (like Jenkins and Docker) are accessible.
Step 4: Install and Configure Jenkins on the Jenkins Instance
Jenkins is the automation server that will orchestrate the CI/CD pipeline.

SSH into the Jenkins EC2 Instance:

bash
Copy code
ssh -i "your-key.pem" ubuntu@<jenkins-instance-ip>
Explanation:

ssh is a protocol for secure remote login. The command connects to your EC2 instance using the private key (your-key.pem) associated with the instance.
Install Java:
Jenkins requires Java to run, so we start by installing the Java Development Kit (JDK).

bash
Copy code
sudo apt update
sudo apt install openjdk-17-jdk -y
Explanation:

sudo apt update updates the package list on the Ubuntu system.
sudo apt install openjdk-17-jdk -y installs the JDK 17, allowing Jenkins to run on the instance.
Install Jenkins:

Add the Jenkins repository key and install Jenkins:
bash
Copy code
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
Explanation:

The curl command retrieves the Jenkins repository key.
The echo command adds the Jenkins repository to the list of sources.
sudo apt-get install jenkins -y installs Jenkins.
Start Jenkins and Verify Installation:

bash
Copy code
sudo systemctl start jenkins
sudo systemctl status jenkins
Explanation:

sudo systemctl start jenkins starts the Jenkins service.
sudo systemctl status jenkins checks if Jenkins is running correctly.
Access Jenkins Web Interface:

Open a web browser and navigate to http://<jenkins-instance-ip>:8080.
Use the initial admin password located in the file /var/lib/jenkins/secrets/initialAdminPassword:
bash
Copy code
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Explanation:

sudo cat displays the content of the initial password file, which you’ll need to unlock the Jenkins setup wizard.
Install Suggested Plugins and Configure Jenkins:

Once you access the Jenkins web interface, follow the setup wizard to install the suggested plugins.
Create an admin user and finish the configuration.
Explanation:

Jenkins plugins extend the functionality of Jenkins. The setup wizard helps in installing essential plugins automatically.
Step 5: Install Docker on the Docker Instance
Docker is the containerization platform that will host your web application in a standardized environment.

SSH into the Docker EC2 Instance:

bash
Copy code
ssh -i "your-key.pem" ubuntu@<docker-instance-ip>
Explanation:

This step connects to the Docker instance in the same way you connected to the Jenkins instance.
Install Docker:

bash
Copy code
sudo apt update
sudo apt install docker.io -y
Explanation:

sudo apt install docker.io -y installs Docker from the Ubuntu package repository.
Add the ubuntu User to the Docker Group:

bash
Copy code
sudo usermod -aG docker ubuntu
Explanation:

usermod -aG docker ubuntu adds the ubuntu user to the docker group, allowing the user to run Docker commands without sudo.
Test the Docker Installation:

bash
Copy code
docker run hello-world
Explanation:

docker run hello-world runs a test container provided by Docker to verify that the installation is working.
Step 6: Configure Jenkins for the CI/CD Pipeline
With Jenkins installed and Docker ready on another instance, the next step is to configure Jenkins to automate the deployment of your application.

Install Jenkins Plugins:

Navigate to Manage Jenkins > Manage Plugins in the Jenkins web interface and install the following plugins:
Git Plugin (for pulling code from GitHub)
Ansible Plugin (for running Ansible playbooks)
Docker Pipeline Plugin (for integrating Docker with Jenkins)
Explanation:

Jenkins plugins enhance the functionality of Jenkins by allowing it to interact with tools like Git, Ansible, and Docker.
Create a New Jenkins Pipeline:

Go to New Item in Jenkins and create a new pipeline project named docker-deploy.
Explanation:

A pipeline in Jenkins defines the series of steps that will be executed to achieve continuous integration and deployment.
Configure the Pipeline to Pull Code from GitHub:

Under Pipeline settings, select Pipeline script from SCM.
Set the SCM to Git and provide the GitHub repository URL.
Add the repository credentials (username and access token).
Explanation:

Pipeline script from SCM instructs Jenkins to use a Jenkinsfile stored in the repository to define the pipeline stages.
Step 7: Set Up Ansible for Configuration Management
Ansible will be used to automate the configuration and deployment of the Docker containers.

Install Ansible on the Jenkins Instance:

bash
Copy code
sudo apt update
sudo apt install ansible -y
Explanation:

ansible is installed on the Jenkins instance so it can be used to manage the Docker instance.
Set Up SSH Keys for Ansible:

Generate an SSH key pair on the Jenkins instance to allow Ansible to connect to the Docker instance without a password:
bash
Copy code
ssh-keygen -t ed25519 -C "jenkins@jenkins"
Copy the public key to the Docker instance:
bash
Copy code
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@<docker-instance-ip>
Explanation:

ssh-keygen generates a new SSH key pair.
ssh-copy-id copies the public key to the authorized_keys file on the Docker instance, enabling passwordless SSH login.
Configure Ansible Hosts File:

Edit the /etc/ansible/hosts file to include the Docker instance:
bash
Copy code
sudo nano /etc/ansible/hosts
Add the following lines:
bash
Copy code
[webserver]
<docker-instance-ip> ansible_python_interpreter=/usr/bin/python3
Explanation:

The Ansible hosts file defines the target machines (like the Docker instance) that Ansible will manage.
ansible_python_interpreter is specified because Ansible uses Python scripts, and this ensures compatibility with the installed Python version on the target machine.
Create an Ansible Playbook to Deploy Docker Containers:

Create a file named deploy.yml in your project directory:
bash
Copy code
nano deploy.yml
Add the following content to the playbook:
yaml
Copy code
- hosts: webserver
  become: yes
  tasks:
    - name: Pull the latest Docker image
      docker_image:
        name: your-docker-image
        source: pull

    - name: Run the Docker container
      docker_container:
        name: mycontainer
        image: your-docker-image
        state: started
        ports:
          - "80:80"
Explanation:

This playbook instructs Ansible to pull the latest Docker image from a repository and run it on the Docker instance, mapping port 80 on the container to port 80 on the instance.
Step 8: Finalize Jenkins Pipeline Script
Integrate the Ansible playbook into the Jenkins pipeline to automate the deployment process.

Edit the Jenkinsfile in Your GitHub Repository:

bash
Copy code
nano Jenkinsfile
Add the following pipeline script:
groovy
Copy code
pipeline {
    agent any
    stages {
        stage('Pull Code') {
            steps {
                git 'https://github.com/yourusername/ansible-jenkins.git'
            }
        }
        stage('Run Ansible Playbook') {
            steps {
                ansiblePlaybook colorized: true, playbook: 'deploy.yml'
            }
        }
    }
}
Explanation:

The Jenkinsfile defines the CI/CD pipeline stages.
The Pull Code stage pulls the latest code from the GitHub repository.
The Run Ansible Playbook stage executes the deploy.yml playbook to deploy the Docker container.
Commit and Push the Jenkinsfile to GitHub:

bash
Copy code
git add Jenkinsfile
git commit -m "Add Jenkinsfile for pipeline"
git push origin main
Explanation:

The Jenkinsfile is version-controlled in the GitHub repository, ensuring that Jenkins always has access to the latest pipeline configuration.
Step 9: Testing and Running the CI/CD Pipeline
Finally, test the entire CI/CD pipeline to ensure everything is functioning correctly.

Trigger the Jenkins Pipeline:

Make a change to your codebase and push it to GitHub.
Jenkins, triggered by the webhook, will start the pipeline automatically.
Monitor the pipeline stages in Jenkins to ensure that the code is pulled, the Ansible playbook is executed, and the Docker container is deployed successfully.
Explanation:

The pipeline automates the entire deployment process, reducing manual intervention and ensuring consistent deployments.
Access the Deployed Application:

Once the pipeline completes, navigate to the Docker instance’s IP address in a web browser to view the deployed application.
Explanation:

The success of the deployment can be verified by accessing the web application served by the Docker container.
Conclusion
By following these steps, you have set up a fully automated CI/CD pipeline using Jenkins, Ansible, Docker, and GitHub on AWS. This pipeline continuously integrates changes from the GitHub repository and deploys the latest version of your application to a Docker container running on an AWS EC2 instance. This setup streamlines the development workflow, reduces the risk of human error, and ensures rapid, consistent deployments.
