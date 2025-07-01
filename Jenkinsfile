pipeline {
  agent any

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'
    NODE_VERSION = '18.20.0'
  }

  stages {
    stage('Setup Environment') {
      steps {
        script {
          // Method 1: Try downloading the .tar.gz version (doesn't require xz)
          sh '''
            # First try with .tar.gz (no xz needed)
            curl -fsSL https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz -o node.tar.gz
            mkdir -p /tmp/nodejs
            tar -xzf node.tar.gz -C /tmp/nodejs --strip-components=1
            export PATH="/tmp/nodejs/bin:$PATH"
            node --version
            npm --version
          '''
          
          // Verify Docker is available
          sh 'docker --version || echo "Docker not found - ensure Docker is installed and the Jenkins user has permissions"'
        }
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
    }

    stage('Build Docker image') {
      steps {
        script {
          sh 'docker build -t ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER} .'
        }
      }
    }

    stage('Push Docker image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "${dockerhublogin}",
          passwordVariable: 'DOCKERHUB_PASSWORD',
          usernameVariable: 'DOCKERHUB_USERNAME'
        )]) {
          sh '''
            echo "${DOCKERHUB_PASSWORD}" | docker login -u ${DOCKERHUB_USERNAME} --password-stdin
            docker push ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER}
          '''
        }
      }
    }
  }

  post {
    always {
      script {
        sh 'command -v docker >/dev/null 2>&1 && docker logout || echo "Docker not available for logout"'
        sh 'rm -rf node_modules || true'
      }
    }
  }
}
