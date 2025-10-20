pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO_URI = '379632383754.dkr.ecr.us-east-1.amazonaws.com/hr-app-management'
        IMAGE_NAME = 'hr-app-management'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/chaitu2306/HR-App-Management.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // use build number as version tag
                    VERSION_TAG = "v${env.BUILD_NUMBER}"
                    sh "docker build -t ${IMAGE_NAME}:${VERSION_TAG} ."
                    sh "docker tag ${IMAGE_NAME}:${VERSION_TAG} ${ECR_REPO_URI}:${VERSION_TAG}"
                    sh "docker tag ${IMAGE_NAME}:${VERSION_TAG} ${ECR_REPO_URI}:latest"
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-ecr-creds', region: "${AWS_REGION}") {
                    script {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URI}"
                    }
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh "docker push ${ECR_REPO_URI}:${VERSION_TAG}"
                    sh "docker push ${ECR_REPO_URI}:latest"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Successfully pushed ${ECR_REPO_URI}:${VERSION_TAG} to AWS ECR"
        }
        always {
            sh "docker system prune -f"
        }
    }
}
