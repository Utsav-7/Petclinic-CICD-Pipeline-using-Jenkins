# Automated CI/CD Pipeline for Secure and Efficient Deployments on Azure VM with Jenkins
## Description
- This document outlines an automated continuous integration and continuous delivery (CI/CD) pipeline for your software project, leveraging Jenkins on an Azure virtual machine (VM). The pipeline emphasizes security and efficiency, encompassing the following stages:

1. **Git Pull:** Retrieves the latest source code from your Git repository.
2. **Maven Compile:** Compiles your code using Maven, ensuring successful compilation before proceeding.
3. **Maven Unit Tests:** Executes unit tests with Maven to detect and address functional issues early.
4. **SonarQube Integration:** Analyzes code quality and security using SonarQube, providing insights into potential vulnerabilities.
5. **OWASP Dependency Check:** Scans dependencies for known vulnerabilities using OWASP Dependency Check, improving overall security posture.
6. **Maven Build:** Builds the final package or artifact after successful tests and code analysis.
7. **Docker Build:** Creates a Docker image encapsulating your application and its dependencies.
8. **Trivy Image Scan:** Performs a security scan on the Docker image using Trivy, identifying potential vulnerabilities.
9. **Docker Push to DockerHub:** Optionally pushes the Docker image to your Dockerhub for secure and centralized storage.
10. **Azure VM Deployment:** Deploys the Docker image to your Azure VM using a preferred deployment method (e.g., Azure CLI, ARM templates).

![pipeline-4 drawio](https://github.com/user-attachments/assets/fa2ce920-8aaa-4636-9c08-65d246b65b89)


## Jenkinsfile (Declarative Pipeline)
```
pipeline {
    agent any

    tools { 
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Utsav-7/Petclinic-cicd.git'
            }
        }
        
        stage('Code Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "mvn test"
            }
        }
        
        
        stage('Sonarqube Anlysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic-cicd \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic-cicd '''
                }
            }
        }
        
        
         stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build Artifact') {
            steps {
                sh "mvn clean install"
            }
        }
        
        
        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '8cc70f5e-0cb6-493b-8322-74ff20547e43', toolName: 'docker') {
                        sh "docker build -t petclinic1 ."
                    }
                }
            }
        }
        
        
        stage('DockerTag & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '8cc70f5e-0cb6-493b-8322-74ff20547e43', toolName: 'docker') {
                        
                        sh "docker tag petclinic1 utsav0712/pet-clinic25:latest"
                        sh "docker push utsav0712/pet-clinic25:latest"
                        
                    }
                }
            }
        }
        
        stage('Trivy Docker Scan') { 
            steps {
                sh "trivy image utsav0712/pet-clinic25:latest"
            }
        }
        
        stage('Deploy using Docker') { 
            steps {
                sh "docker run -d --name pet1 -p 8082:8082 utsav0712/pet-clinic25:latest"
            }
        }
        
    }
}
```
## Snapshots: 
### Azure Virtual Machine:

![Screenshot from 2024-07-18 22-17-06](https://github.com/user-attachments/assets/0f441469-39e9-4aff-bfbd-e49b3b8183eb)

![Screenshot from 2024-07-18 22-18-38](https://github.com/user-attachments/assets/7d2e1eba-9ec6-4bf4-80fc-de9807410c7c)

### Dockerhub:
![Screenshot from 2024-07-18 22-16-34](https://github.com/user-attachments/assets/35136c8a-d16f-4d3d-a5c1-9f8077f94956)

### Jenkins Pipeline: 
![Screenshot from 2024-07-18 22-14-55](https://github.com/user-attachments/assets/0e931af2-51d4-4bfa-9317-433b7665c54a)
![Screenshot from 2024-07-18 18-42-43](https://github.com/user-attachments/assets/c0cbe71d-c5e9-4514-9b40-f7b62042f667)

### Sonarqube Analysis: 
![Screenshot from 2024-07-18 22-15-38](https://github.com/user-attachments/assets/5207f261-513e-4d39-a258-c8867ff4b6c0)

### Application access:
<img width="1042" alt="petclinic-screenshot" src="https://cloud.githubusercontent.com/assets/838318/19727082/2aee6d6c-9b8e-11e6-81fe-e889a5ddfded.png">

## Tools and Versions:
- **Jenkins:** Latest LTS version (currently 2.390.2 as of July 19, 2024)
- **Maven:** 3.8.6 (Recommended version for JDK 17 compatibility)
- **SonarQube:** Latest LTS version (currently 9.3.3 LTS as of July 19, 2024)
- **OWASP Dependency Check:** Latest version compatible with Maven 3.8.6 (currently 7.1.0)
- **JDK:** 17.0.4 (LTS version)
- **Docker:** Latest version (currently 20.10.22 as of July 19, 2024)
- **Trivy:** 0.39.0 (Recommended version for stability and feature set)
