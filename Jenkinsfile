pipeline {
agent any


stages {

    stage('Checkout') {
        steps {
            checkout scm
        }
    }

    stage('Build Docker Images') {
        steps {
            bat 'docker compose build'
        }
    }

    stage('Security Scan - Backend') {
        steps {
            bat 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image ci-cd-assessment-backend:latest'
        }
    }

    stage('Security Scan - Frontend') {
        steps {
            bat 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image ci-cd-assessment-frontend:latest'
        }
    }

    stage('Deploy') {
        steps {
            bat 'docker compose up -d'
        }
    }

    stage('Health Check') {
        steps {
            powershell 'Start-Sleep -Seconds 10'
            bat 'curl -f http://localhost:8080'
            bat 'curl -f --retry 5 --retry-delay 3 http://127.0.0.1:5001/health'
        }
    }

    stage('Docker Image Cleanup') {
        steps {
            bat 'docker image prune -f'
        }
    }

    stage('Rollback') {
        steps {
            bat 'docker compose down'
            bat 'docker compose up -d'
        }
    }
}


}
