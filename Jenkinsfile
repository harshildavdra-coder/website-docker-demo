pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'website-docker-demo'
        AWS_ACCOUNT_ID = '939365918175'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
        LATEST_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest"
        DEPLOY_SERVER = '13.232.5.50'
        SSH_USER = 'ubuntu'          // Or 'jenkins', depending on your EC2 user
        SSH_KEY = '/var/lib/jenkins/.ssh/deploy-ec2.pem'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/harshildavdra-coder/website-docker-demo.git',
                        credentialsId: 'deploy-ec2-key'
                    ]]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t website-docker-demo .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh """
                    docker tag website-docker-demo ${IMAGE_URI}
                    docker tag website-docker-demo ${LATEST_URI}
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                    docker push ${IMAGE_URI}
                    docker push ${LATEST_URI}
                """
            }
        }

        stage('Deploy to EC2') {
    steps {
        sshagent(['deploy-ec2-key']) {
            sh """
                ssh -o StrictHostKeyChecking=no ubuntu@${DEPLOY_SERVER} \\
                "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com && \\
                docker pull ${LATEST_URI} && \\
                docker stop website-demo || true && \\
                docker rm website-demo || true && \\
                docker run -d --name website-demo -p 80:80 ${LATEST_URI}"
            """
        }
    }
}

    post {
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
