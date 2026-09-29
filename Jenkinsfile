pipeline {
agent any

```
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

    stage('Deploy') {
        steps {
            bat 'docker compose up -d'
        }
    }

    stage('Health Check') {
        steps {
            bat 'curl -f http://localhost:8080'
            bat 'curl -f http://localhost:5001'
        }
    }
}
```

}
