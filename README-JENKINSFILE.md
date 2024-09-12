### CloudSheger DevOps Engineers: CI/CD Pipeline Project

This project guides you through building a CI/CD pipeline for a complete deployment process using various DevOps tools. You will create an automated pipeline that integrates Jenkins, GitHub, Artifactory, X-Ray, SonarQube, and Ansible for deploying applications to UAT and Production servers. This project will provide hands-on experience with building infrastructure using Vagrant and implementing CI/CD practices in a real-world DevOps workflow.

---

## Project Overview

### Key Objectives:
1. **Infrastructure Setup with Vagrant**:
   - Provision two virtual servers using Vagrant:
     - **UAT Server**: For deploying and testing applications.
     - **Production Server**: For final application deployment.
   - Each server will mimic real-world environments to ensure smooth development and deployment processes.

2. **CI/CD Pipeline with Jenkins**:
   - Jenkins will be integrated with several key tools:
     - **GitHub**: For version control and tracking changes in the project.
     - **Artifactory & X-Ray**: To manage and scan application artifacts for security vulnerabilities.
     - **SonarQube**: To analyze code quality and enforce best practices.
     - **Ansible**: To automate deployment to both UAT and Production servers.

### Tools & Technologies Used:
- **Vagrant**: Infrastructure provisioning
- **Jenkins**: CI/CD automation tool
- **GitHub**: Version control system
- **Artifactory**: Artifact repository manager
- **X-Ray**: Security and compliance scanning
- **SonarQube**: Static code analysis tool
- **Ansible**: Configuration management and deployment

---

## Step-by-Step Guide

### 1. Build Infrastructure with Vagrant

Set up two virtual servers (UAT and Production) using Vagrant for deployment.

**Vagrantfile Example**:
```ruby
Vagrant.configure("2") do |config|

  config.vm.define "UAT" do |uat|
    uat.vm.box = "ubuntu/bionic64"
    uat.vm.hostname = "uat-server"
    uat.vm.network "private_network", ip: "192.168.56.10"
    uat.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 2
    end
  end

  config.vm.define "PRODUCTION" do |prod|
    prod.vm.box = "ubuntu/bionic64"
    prod.vm.hostname = "prod-server"
    prod.vm.network "private_network", ip: "192.168.56.20"
    prod.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end
  end
end
```

Run the following command to bring up the infrastructure:

```bash
vagrant up
```

### 2. Jenkins Pipeline Overview

A Jenkins pipeline will automate the following stages:
- **Git Checkout**: Clones the repository from GitHub.
- **OWASP FS Scan**: Scans the code for security vulnerabilities using OWASP Dependency Check.
- **SonarQube Analysis**: Analyzes the code using SonarQube for code quality checks.
- **Build and Test**: Installs dependencies, builds the backend and frontend, and performs testing.
- **Deploy to Containers**: Deploys the application using Docker Compose.

---

## Jenkinsfile Template

This Jenkinsfile defines the steps for building and deploying the application using a CI/CD pipeline.

```groovy
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jaiswaladi246/fullstack-bank.git'
            }
        }
        
        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./app/backend --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Bank -Dsonar.projectKey=Bank"
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Build Backend') {
            steps {
                dir('app/backend') {
                    sh 'npm install'
                }
            }
        }
        
        stage('Build Frontend') {
            steps {
                dir('app/frontend') {
                    sh 'npm install'
                }
            }
        }
        
        stage('Deploy to Container') {
            steps {
                sh 'npm run compose:up -d'
            }
        }
    }
}
```

### Explanation of the Jenkins Pipeline:
1. **Git Checkout**:
   - Jenkins will clone the GitHub repository to ensure the pipeline is working on the latest code.

2. **OWASP FS Scan**:
   - The OWASP Dependency Check scans the backend code for vulnerabilities in dependencies.

3. **SonarQube Analysis**:
   - The pipeline integrates with SonarQube to perform static code analysis and generate a report on code quality.

4. **Install Dependencies**:
   - Jenkins installs the required dependencies for the application using `npm`.

5. **Build Backend and Frontend**:
   - Builds the backend and frontend components of the application.

6. **Deploy to Container**:
   - Once the application is built, Jenkins uses Docker Compose to deploy the application to a container.

---

### 3. Security & Quality Scans

This pipeline includes two major scanning stages:
- **OWASP FS Scan**: Used for scanning for security vulnerabilities in the backend code dependencies.
- **SonarQube**: Performs static code analysis to ensure code quality and best practices.

### 4. Deployment to UAT and Production Servers

To deploy the built application on the UAT and Production servers:
- Use Ansible playbooks to handle the deployment of Docker containers to the servers.
- Ansible will pull the latest Docker images from Artifactory and deploy them to the appropriate servers.

### Ansible Playbook Example:

```yaml
---
- name: Deploy to UAT Server
  hosts: uat
  tasks:
    - name: Pull Docker image from Artifactory
      docker_image:
        name: "{{ artifactory_url }}/{{ image_name }}:latest"
    
    - name: Start Docker container
      docker_container:
        name: bank-app
        image: "{{ artifactory_url }}/{{ image_name }}:latest"
        state: started
        restart_policy: always
```

---

### 5. Additional Instructions

- **Artifactory & X-Ray Integration**:
  - Ensure your Jenkins pipeline pushes built artifacts (JARs, Docker images, etc.) to JFrog Artifactory. Artifactory's X-Ray component can scan these artifacts for vulnerabilities.
  
- **SonarQube**:
  - SonarQube should be configured to enforce code quality rules and block deployments if issues are found.

- **UAT vs Production**:
  - Use different branches and deployment strategies (such as blue-green or canary deployments) to control what gets pushed to UAT and Production environments.

---

## Conclusion

By the end of this project, CloudSheger DevOps engineers will have built a fully functional CI/CD pipeline using Jenkins and integrated tools such as GitHub, Artifactory, SonarQube, and Ansible. This pipeline will enable efficient, secure, and automated deployments to UAT and Production environments.

---

### Next Steps:
- Clone this repository, build the infrastructure using Vagrant, and set up Jenkins with the provided pipeline.
- Modify the Jenkinsfile to suit your specific deployment needs (such as different project URLs).
- Experiment with security and code quality tools to ensure your pipeline meets enterprise-level standards.

### Reference 

Jenkinsfile : https://github.com/shegerbootcamp/docker-lab/blob/master/ci/Jenkinsfile.dev

Project : https://github.com/shegerbootcamp/docker-lab.git


