pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-southeast-1'
        ECR_REPO_BACKEND = 'backend'
        ECR_REPO_FRONTEND = 'frontend'
        AWS_ACCOUNT_ID = '476300133240'
        IMAGE_TAG = "latest-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/lynguyenhuu/sd3971_msa.git'
            }
        }
        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    script {
                        docker.build("backend:${IMAGE_TAG}", "-f ${env.WORKSPACE}/src/backend/Dockerfile .")

                        sh "docker tag backend:${IMAGE_TAG} ${ECR_REPO_BACKEND}:${IMAGE_TAG}"
                    }
                }
            }
        }
        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    script {
                        docker.build("frontend:${IMAGE_TAG}", "-f ${env.WORKSPACE}/src/frontend/Dockerfile .")
                        
                        sh "docker tag frontend:${IMAGE_TAG} ${ECR_REPO_FRONTEND}:${IMAGE_TAG}"
                    }
                }
            }
        }
        stage('Login to ECR') {
            steps {
                script {
                    withCredentials([aws(credentialsId: "${AWS_ACCOUNT_ID}", region: "${AWS_REGION}")]) {
                        sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                    }
                }
            }
        }
        stage('Push Backend Docker Image to ECR') {
            steps {
                script {
                    sh "docker push ${ECR_REPO_BACKEND}:${IMAGE_TAG}"
                }
            }
        }
        stage('Push Frontend Docker Image to ECR') {
            steps {
                script {
                    sh "docker push ${ECR_REPO_FRONTEND}:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                script {
                    // Ensure Kubernetes CLI is available and configured properly
                    sh 'kubectl version --client'

                    // Apply the Kubernetes deployment and service files
                    sh '''
                    kubectl apply -f k8s/backend-deployment.yaml
                    kubectl apply -f k8s/backend-service.yaml
                    kubectl apply -f k8s/frontend-deployment.yaml
                    kubectl apply -f k8s/frontend-service.yaml
                    '''
                }
            }
        }
    }
}
