pipeline {
    agent any
    tools{
        jdk  'jdk11'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/prakashbelote/Python-Webapp.git'            }
        }
        stage('OWASP Scan') {
            steps {
                 dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        stage('Trivi File Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Python-WebApp \
                   -Dsonar.projectKey=Python-WebApp '''
               }
            }
        }
        stage('Docker Image Build') {
            steps {
                script{
                    withDockerRegistry(credentialsId:  '70e0ce70-b8c5-4aba-b853-4f74ced373a9', toolName: 'docker') {
                        
                      sh "make image"
                    }
                }
            }
        }
        stage('Trivi Image Scan') {
            steps {
                sh "trivy image  prakashbelote/python-web:latest"
            }
        }
        stage('Docker Image Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId:  '70e0ce70-b8c5-4aba-b853-4f74ced373a9', toolName: 'docker') {
                        
                      sh "make push"
                    }
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                script{
                    withDockerRegistry(credentialsId:  '70e0ce70-b8c5-4aba-b853-4f74ced373a9', toolName: 'docker') {
                        
                      sh "docker run -d -p 5000:5000 prakashbelote/python-web:latest"
                    }
                }
            }
        }
        
    }
}
