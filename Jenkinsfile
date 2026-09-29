
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
    }
}

⚠️ **Important:** Copy only the Jenkinsfile content. Do **not** copy the ``` markers.

### Now save and push

Run in:

**PowerShell → `C:\Users\DELL\Documents\ci-cd-assessment`**

powershell
git add Jenkinsfile
git commit -m "Add Docker image cleanup"
git push origin main
```

Then go to **Jenkins → `ci-cd-assessment` → Build Now**.

We want to see:

**Docker Image Cleanup → SUCCESS** ✅

After that, we'll do the **Rollback step**.
