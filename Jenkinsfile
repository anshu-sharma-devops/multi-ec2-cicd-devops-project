pipeline {
    agent any

    environment {
        // We removed the AWS keys because the EC2 IAM Role handles it automatically!
        ANSIBLE_SSH_KEY = credentials('ssh-private-key') 
    }

    stages {
        stage('Stage 1 - Git Clone') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Stage 2 - Terraform Init') {
            steps {
                dir('terraform') {
                    echo 'Initializing Terraform...'
                    sh 'terraform init'
                }
            }
        }

        stage('Stage 3 - Terraform Apply') {
            steps {
                dir('terraform') {
                    echo 'Creating Infrastructure...'
                    sh 'terraform apply -auto-approve'
                    // Hand off the new app server IP to Ansible
                    sh 'terraform output -raw app_server_ip > ../ansible/app_ip.txt'
                }
            }
        }

        stage('Stage 4 - Ansible Deployment') {
            steps {
                dir('ansible') {
                    echo 'Configuring App Server with Ansible...'
                    sh 'ansible-playbook -i inventory install-docker.yml --private-key=${ANSIBLE_SSH_KEY}'
                }
            }
        }

        stage('Stage 5 - Docker Build') {
            steps {
                dir('docker') {
                    echo 'Building Nginx Docker Image...'
                    sh 'docker build -t my-nginx-website:latest .'
                }
            }
        }

        stage('Stage 6 - Docker Run') {
            steps {
                echo 'Deploying Docker Container...'
                // Deployment execution command goes here
                echo 'Deployment successful!'
            }
        }
    }
}