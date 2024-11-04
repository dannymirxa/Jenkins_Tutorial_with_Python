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
        stage('Prune System') {
            steps {
                sh 'docker system prune -a'
            }
        }
        stage('Test') {
            steps {
                script {
                    def dockerImage
                    try {
                        dockerImage = docker.build('fastapi-test:latest', '-f pytest-dockerfile .')
                        dockerImage.inside {
                            sh 'pytest test_main.py'
                        }
                    } catch (Exception e) {
                        echo "Tests failed: ${e}"
                        throw e
                    } finally {
                        sh 'docker image rm "fastapi-test" -f'
                        echo 'Remove Docker Image "fastapi-test"'
                    }
                }
            }
        }
        stage('Deploy FastAPI app') {
            steps {
                sh 'docker compose -f fastapi-compose.yaml up -d'
            }
        }
    }
}
