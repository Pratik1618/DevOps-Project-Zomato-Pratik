pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'zomato-app'
    }
    stages {
        stage('Get AWS Account ID') {
            steps {
                script {
                    // Ask AWS for the Account ID dynamically
                    env.AWS_ACCOUNT_ID = sh(script: 'aws sts get-caller-identity --query Account --output text', returnStdout: true).trim()
                    env.ECR_REGISTRY = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Zomato Docker Image..."
                    sh "docker build -t ${ECR_REPO}:latest ."
                }
            }
        }
        stage('Push to ECR') {
            steps {
                script {
                    echo "Authenticating with AWS ECR..."
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                    
                    echo "Tagging and Pushing Image to ECR..."
                    sh "docker tag ${ECR_REPO}:latest ${ECR_REGISTRY}/${ECR_REPO}:latest"
                    sh "docker push ${ECR_REGISTRY}/${ECR_REPO}:latest"
                }
            }
        }
    }
}
