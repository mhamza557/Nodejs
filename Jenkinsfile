pipeline {
  agent {
    docker {
      image 'node:18-alpine'  // Smaller footprint than regular node image
      args '--user root'  // Ensure proper permissions
      reuseNode true  // Reuse the workspace on the host
    }
  }

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'
    CI = 'true'  // Standard CI environment variable
  }

  stages {
    stage('Checkout & Setup') {
      steps {
        checkout scm
        sh 'node --version && npm --version'
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm ci --prefer-offline --audit false'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
    }

    stage('Build Docker image') {
      agent {
        docker {
          image 'docker:24.0-cli-alpine3.18'  // Use Docker-in-Docker
          args '-v /var/run/docker.sock:/var/run/docker.sock --user root'
          reuseNode true
        }
      }
      steps {
        script {
          sh 'docker buildx create --use'
          sh """
            docker buildx build \
              --platform linux/amd64 \
              -t ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER} \
              -t ${DOCKERHUB_REGISTRY}:latest \
              --push .
          """
        }
      }
    }
  }

  post {
    always {
      script {
        sh 'docker logout || true'
        cleanWs()  // Clean up workspace
      }
    }
    success {
      slackSend color: 'good', message: "Build ${BUILD_NUMBER} succeeded"
    }
    failure {
      slackSend color: 'danger', message: "Build ${BUILD_NUMBER} failed"
    }
  }
}
