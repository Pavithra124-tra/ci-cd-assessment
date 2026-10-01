
pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '382597877772'

        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        FRONTEND_REPO = 'ci-cd-assessment-frontend'
        BACKEND_REPO = 'ci-cd-assessment-backend'

        FRONTEND_IMAGE = "${ECR_REGISTRY}/${FRONTEND_REPO}:latest"
        BACKEND_IMAGE = "${ECR_REGISTRY}/${BACKEND_REPO}:latest"

        ECS_CLUSTER = 'ci-cd-assessment-cluster'
        FRONTEND_SERVICE = 'ci-cd-assessment-frontend-service'
        BACKEND_SERVICE = 'ci-cd-assessment-backend-service'

        ALB_URL = 'http://ci-cd-assessment-alb-1756184904.us-east-1.elb.amazonaws.com'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
                sh 'docker compose build'
            }
        }

        stage('Security Scan - Backend') {
            steps {
                echo 'Scanning backend image with Trivy...'
                sh 'trivy image ci-cd-assessment-backend:latest'
            }
        }

        stage('Security Scan - Frontend') {
            steps {
                echo 'Scanning frontend image with Trivy...'
                sh 'trivy image ci-cd-assessment-frontend:latest'
            }
        }

        stage('Login to ECR') {
            steps {
                echo 'Logging in to Amazon ECR...'
                sh '''
                    aws ecr get-login-password \
                    --region $AWS_REGION | \
                    docker login \
                    --username AWS \
                    --password-stdin $ECR_REGISTRY
                '''
            }
        }

        stage('Tag Images') {
            steps {
                echo 'Tagging Docker images for ECR...'
                sh '''
                    docker tag \
                    ci-cd-assessment-frontend:latest \
                    $FRONTEND_IMAGE

                    docker tag \
                    ci-cd-assessment-backend:latest \
                    $BACKEND_IMAGE
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                echo 'Pushing Docker images to ECR...'
                sh '''
                    docker push $FRONTEND_IMAGE
                    docker push $BACKEND_IMAGE
                '''
            }
        }

        stage('Deploy Frontend to ECS') {
            steps {
                echo 'Deploying frontend to ECS...'
                sh '''
                    aws ecs update-service \
                    --cluster $ECS_CLUSTER \
                    --service $FRONTEND_SERVICE \
                    --force-new-deployment \
                    --region $AWS_REGION
                '''
            }
        }

        stage('Deploy Backend to ECS') {
            steps {
                echo 'Deploying backend to ECS...'
                sh '''
                    aws ecs update-service \
                    --cluster $ECS_CLUSTER \
                    --service $BACKEND_SERVICE \
                    --force-new-deployment \
                    --region $AWS_REGION
                '''
            }
        }

        stage('Wait for ECS Deployment') {
            steps {
                echo 'Waiting for ECS services to stabilize...'

                sh '''
                    aws ecs wait services-stable \
                    --cluster $ECS_CLUSTER \
                    --services $FRONTEND_SERVICE \
                    --region $AWS_REGION

                    aws ecs wait services-stable \
                    --cluster $ECS_CLUSTER \
                    --services $BACKEND_SERVICE \
                    --region $AWS_REGION
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking application through ALB...'

                sh '''
                    sleep 10
                    curl -f $ALB_URL/
                '''
            }
        }

        stage('Docker Image Cleanup') {
            steps {
                echo 'Cleaning unused Docker images...'
                sh 'docker image prune -f'
            }
        }
    }

    post {
        success {
            echo '========================================='
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY'
            echo '========================================='
            echo 'Application is deployed to AWS ECS.'
        }

        failure {
            echo '========================================='
            echo 'CI/CD PIPELINE FAILED'
            echo '========================================='
            echo 'Check the failed stage in the Jenkins console.'
        }
    }
}

