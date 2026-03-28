pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '939365918175.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo'
        IMAGE_NAME = 'website-docker-demo'
        EC2_USER = 'ubuntu'
        EC2_HOST = '13.232.5.50' // replace with your EC2 public IP
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/harshildavdra-coder/website-docker-demo.git',
                    credentialsId: 'deploy-ec2-key'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh "docker tag ${IMAGE_NAME} ${ECR_REPO}:latest"
            }
        }

        stage('Login to ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${ECR_REPO}:latest"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ubuntu-ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                        docker pull ${ECR_REPO}:latest &&
                        docker stop ${IMAGE_NAME} || true &&
                        docker rm ${IMAGE_NAME} || true &&
                        docker run -d --name ${IMAGE_NAME} -p 80:80 ${ECR_REPO}:latest
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished!'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
