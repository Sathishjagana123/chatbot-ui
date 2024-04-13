pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node19'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Sathishjagana123/chatbot-ui.git'
            }
        }
        stage('install dependencies') {
            steps {
                sh "npm install"
            }
        }
       stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Chatbot \
                    -Dsonar.projectKey=Chatbot '''
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
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.json"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh "docker build -t chatbot ."
                        sh "docker tag chatbot satishjagana/chatbot:latest"
                        sh "docker push satishjagana/chatbot:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image satishjagana/chatbot:latest > trivy.json" 
            }
        }
        stage("Remove container"){
            steps{
                sh "docker stop chatbot || true"
                sh "docker rm chatbot  || true"
                
            }
        }
        stage ("Deploy to container"){
            steps{
                sh 'docker run -d --name chatbot -p 3000:3000 satishjagana/chatbot:latest'
            }
        }
    }
}
