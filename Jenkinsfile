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
                sh 'docker build -t devops-web:v4 ./docker'
            }
        }

        stage('Verify Image') {
            steps {
                sh 'docker images'
            }
        }

        stage('Deploy to App Server') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@65.0.199.227 "
                sudo docker stop webapp || true
                sudo docker rm webapp || true
                sudo docker run -d --name webapp -p 80:80 devops-web:v4
                "
                '''
            }
        }
    }
}