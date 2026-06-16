pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t devops-web:latest ./docker'
            }
        }

        stage('List Images') {
            steps {
                sh 'docker images'
            }
        }
    }
}