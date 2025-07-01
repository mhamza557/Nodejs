pipeline {
  agent any

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'
    NODE_VERSION = '18.x'  // Using Node.js 18
  }

  stages {
    stage('Setup Environment') {
      steps {
        script {
          // Install Node.js 18 without sudo
          sh '''
            curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION} | bash -
            apt-get update && apt-get install -y nodejs
            corepack enable  # Enable yarn/pnpm if needed
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
        sh 'npm ci'  // Using npm ci for cleaner, reproducible installs
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
          // Build with BuildKit for better performance
          sh 'DOCKER_BUILDKIT=1 docker build --pull --no-cache -t ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER} .'
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
        // Clean up Docker login only if Docker is available
        sh 'command -v docker >/dev/null 2>&1 && docker logout || echo "Docker not available for logout"'
        
        // Optional: Clean up node_modules to save workspace space
        sh 'rm -rf node_modules || true'
      }
    }
  }
}
