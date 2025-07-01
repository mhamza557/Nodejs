pipeline {
  agent {
    docker {
      image 'node:18-alpine'
      args '--user root'
      reuseNode true
    }
  }

  // Only include tools section if you want to use Jenkins-managed NodeJS
  // tools {
  //   nodejs 'NodeJS-18' // Must match exactly what you configured in Global Tools
  // }

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'
    CI = 'true'
    NODE_ENV = 'production'
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
          image 'docker:24.0-cli-alpine3.18'
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
              --push .
          """
        }
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
      cleanWs()
    }
  }
}
