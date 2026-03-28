pipeline {
    agent any

    environment {
        ECR_REGISTRY = "939365918175.dkr.ecr.ap-south-1.amazonaws.com"
        IMAGE_NAME = "website-docker-demo"
        IMAGE_TAG  = "latest"
        EC2_USER   = "ubuntu"
        EC2_IP     = "13.232.5.50"
        SSH_KEY    = "/var/lib/jenkins/.ssh/deploy-ec2.pem"
        AWS_REGION = "ap-south-1"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh """
                    docker tag ${IMAGE_NAME} ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker tag ${IMAGE_NAME} ${ECR_REGISTRY}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                    docker push ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['deploy-ec2-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} \\
                        "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY} && \\
                        docker pull ${ECR_REGISTRY}/${IMAGE_NAME}:latest && \\
                        docker stop website-demo || true && \\
                        docker rm website-demo || true && \\
                        docker run -d --name website-demo -p 80:80 ${ECR_REGISTRY}/${IMAGE_NAME}:latest"
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
