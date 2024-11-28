pipeline {
    agent any
    tools {
        maven '3.9.9'
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('liberosis8803-dockerhub')
        SONARQUBE_SERVER = 'http://localhost:9000'
        SONARQUBE_TOKEN = credentials('sonarqube-secret-2')
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/SleepySouls/DevOps-Practices-Lab2-Assignment-3.git']]
                )
            }
        }
        
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-scanner') {}
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=jenkins-scanner \
                        -Dsonar.projectName='jenkins-scanner' \
                        -Dsonar.host.url=$SONARQUBE_SERVER \
                        -Dsonar.login=$SONARQUBE_TOKEN
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t liberosis8803/devopslab2:latest .'
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                sh """
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push liberosis8803/devopslab2:latest
                """
            }
        }

        stage('Deploy to Kubernetes'){
            steps{
                script{
                    kubernetesDeploy (configs: 'k8s-deployment.yaml',
                    kubeconfigId: 'k8sconfiguration')
                    }
            }
        }
    }
