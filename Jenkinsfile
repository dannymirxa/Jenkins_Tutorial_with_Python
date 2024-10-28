pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[credentialsId: 'fastapi-app', url: 'https://github.com/dannymirxa/FastAPI-App-Example.git']])
            }
        }
        stage('Build') {
            steps {
                git branch: 'main', credentialsId: 'fastapi-app', url: 'https://github.com/dannymirxa/FastAPI-App-Example.git'
            }
        }
        stage('Test') {
            steps {
                script {
                    docker.build('fastapi-test:latest', '-f pytest-dockerfile .').inside {
                        sh 'pytest test_main.py'
                    }
                }
            }
        }
        
    }
}
