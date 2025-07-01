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
          // Install Node.js without sudo
          sh '''
            curl -fsSL https://deb.nodesource.com/setup_16.x | bash -
            apt-get update && apt-get install -y nodejs
            node --version
            npm --version
          '''
          
          // Verify Docker is available or install it
          sh 'docker --version || echo "Docker not found, will need to install"'
        }
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm install'
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
          credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
          passwordVariable: 'DOCKERHUB_PASSWORD',
          usernameVariable: 'DOCKERHUB_USERNAME'
        )]) {
          sh 'echo "${DOCKERHUB_PASSWORD}" | docker login -u ${DOCKERHUB_USERNAME} --password-stdin'
          sh 'docker push ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER}'
        }
      }
    }
  }

  post {
    always {
      script {
        // Only try to logout if docker is available
        sh 'docker --version && docker logout || echo "Docker not available"'
      }
    }
  }
}
