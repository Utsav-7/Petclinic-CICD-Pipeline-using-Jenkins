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
