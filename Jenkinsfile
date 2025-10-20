pipeline {
    agent any

    environment {
        // Basic configuration
        AWS_REGION = 'us-east-1'
        ECR_REPO_URI = '379632383754.dkr.ecr.us-east-1.amazonaws.com/hr-app-management'
        IMAGE_NAME = 'hr-app-management'
        DOCKER_HUB_REPO = 'chaitu2003/hr-app-management'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'master', url: 'https://github.com/chaitu2306/HR-App-Management.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build tag using Jenkins build number
                    VERSION_TAG = "v${env.BUILD_NUMBER}"

                    // Build Docker image
                    sh "docker build -t ${IMAGE_NAME}:${VERSION_TAG} ."

                    // Tag for Docker Hub
                    sh "docker tag ${IMAGE_NAME}:${VERSION_TAG} ${DOCKER_HUB_REPO}:${VERSION_TAG}"
                    sh "docker tag ${IMAGE_NAME}:${VERSION_TAG} ${DOCKER_HUB_REPO}:latest"

                    // Tag for AWS ECR
                    sh "docker tag ${IMAGE_NAME}:${VERSION_TAG} ${ECR_REPO_URI}:${VERSION_TAG}"
                    sh "docker tag ${IMAGE_NAME}:${VERSION_TAG} ${ECR_REPO_URI}:latest"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ${DOCKER_HUB_REPO}:${VERSION_TAG}"
                        sh "docker push ${DOCKER_HUB_REPO}:latest"
                    }
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    script {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URI}"
                    }
                }
            }
        }

        stage('Push to AWS ECR') {
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
            echo "✅ Successfully built and pushed images:"
            echo "   - Docker Hub: ${DOCKER_HUB_REPO}:${VERSION_TAG}"
            echo "   - AWS ECR: ${ECR_REPO_URI}:${VERSION_TAG}"
        }
        always {
            // Optional cleanup to save space
            sh "docker system prune -f"
        }
    }
}
