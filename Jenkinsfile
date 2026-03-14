pipeline {
    agent { label 'ecs-slave' }

    environment {
        AWS_ACCOUNT_ID = '596059883112'  
        AWS_REGION     = 'us-east-1'        
        CLUSTER_NAME   = 'Three-tier-App' 
        
        BACKEND_REPO   = 'backend-node'
        FRONTEND_REPO  = 'frontend-react'

        BASE_VERSION   = '1' 
        APP_VERSION    = "v${BASE_VERSION}.${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('ECR Login') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                }
            }
        }

        stage('Build & Push Backend') {
            steps {
                script {
                    def backendUri = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_REPO}"
                    
                    sh "docker build -t ${BACKEND_REPO}:${APP_VERSION} ./backend"
                    sh "docker tag ${BACKEND_REPO}:${APP_VERSION} ${backendUri}:${APP_VERSION}"
                    sh "docker tag ${BACKEND_REPO}:${APP_VERSION} ${backendUri}:latest"
                    
                    sh "docker push ${backendUri}:${APP_VERSION}"
                    sh "docker push ${backendUri}:latest"
                }
            }
        }

        stage('Build & Push Frontend') {
            steps {
                script {
                    def frontendUri = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_REPO}"
                
                    sh "docker build -t ${FRONTEND_REPO}:${APP_VERSION} ./frontend"
                    sh "docker tag ${FRONTEND_REPO}:${APP_VERSION} ${frontendUri}:${APP_VERSION}"
                    sh "docker tag ${FRONTEND_REPO}:${APP_VERSION} ${frontendUri}:latest"
                    
                    sh "docker push ${frontendUri}:${APP_VERSION}"
                    sh "docker push ${frontendUri}:latest"
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    sh "aws ecs update-service --cluster ${CLUSTER_NAME} --service backend-service-new --force-new-deployment --region ${AWS_REGION}"
                    sh "aws ecs update-service --cluster ${CLUSTER_NAME} --service frontend-service-new --force-new-deployment --region ${AWS_REGION}"
                }
            }
        }
    }
    
    post {
        always {
            sh "docker image prune -f"
        }
    }
}