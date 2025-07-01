pipeline {
  agent any

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'
  }

  stages {
    stage('Setup Environment') {
      steps {
        script {
          // Alternative Node.js installation method that doesn't require apt-get
          sh '''
            # Download and extract Node.js 18 binaries
            curl -fsSL https://nodejs.org/dist/v18.20.0/node-v18.20.0-linux-x64.tar.xz -o node.tar.xz
            mkdir -p /tmp/nodejs
            tar -xJf node.tar.xz -C /tmp/nodejs --strip-components=1
            export PATH="/tmp/nodejs/bin:$PATH"
            
            # Verify installation
            node --version
            npm --version
          '''
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
          sh 'docker --version || echo "Docker not available"'
          sh 'docker build -t ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER} .'
        }
      }
    }

    stage('Push Docker image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
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
