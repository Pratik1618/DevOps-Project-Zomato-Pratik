pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'zomato-app'
        EKS_CLUSTER = 'zomato-cluster'
    }
    stages {
        stage('Get AWS Account ID') {
            steps {
                script {
                    env.AWS_ACCOUNT_ID = sh(script: 'aws sts get-caller-identity --query Account --output text', returnStdout: true).trim()
                    env.ECR_REGISTRY = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                    
                    // Notice we use BUILD_NUMBER now instead of 'latest'
                    env.FULL_IMAGE_URL = "${env.ECR_REGISTRY}/${ECR_REPO}:v${BUILD_NUMBER}"
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Zomato Docker Image v${BUILD_NUMBER}..."
                    sh "docker build -t ${ECR_REPO}:v${BUILD_NUMBER} ."
                }
            }
        }
        stage('Push to ECR') {
            steps {
                script {
                    echo "Authenticating with AWS ECR..."
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                    
                    echo "Tagging and Pushing Image to ECR..."
                    sh "docker tag ${ECR_REPO}:v${BUILD_NUMBER} ${FULL_IMAGE_URL}"
                    sh "docker push ${FULL_IMAGE_URL}"
                }
            }
        }
        stage('Deploy to Staging') {
            steps {
                script {
                    echo "Authenticating with EKS Cluster..."
                    sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}"
                    
                    echo "Injecting Dynamic Docker Image URL into Staging YAML..."
                    sh "sed -i 's|<IMAGE_URL>|${FULL_IMAGE_URL}|g' staging.yml"
                    
                    echo "Deploying to Staging Namespace..."
                    sh "kubectl apply -f staging.yml"
                }
            }
        }
        stage('Approval Gate') {
            steps {
                input message: "Staging deployment complete. Does it look good?", ok: "Deploy to Production"
            }
        }
        stage('Deploy to Production') {
            steps {
                script {
                    echo "Injecting Dynamic Docker Image URL into Production YAML..."
                    sh "sed -i 's|<IMAGE_URL>|${FULL_IMAGE_URL}|g' prod.yml"
                    
                    echo "Deploying to Production Namespace..."
                    sh "kubectl apply -f prod.yml"
                }
            }
        }
    }
}
