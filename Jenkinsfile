pipeline {
  agent any

  // Remove the 'tools' section if NodeJS plugin is not installed
  // If you have NodeJS plugin installed, ensure 'node' is configured in Jenkins Global Tools
  // tools {
  //   nodejs 'node'  // 'node' must match a NodeJS installation name in Jenkins config
  // }

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'  // Fixed variable name in withCredentials
  }

  stages {
    stage('Setup Node.js') {
      steps {
        // Alternative if NodeJS plugin isn't installed
        sh '''
          curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
          sudo apt-get install -y nodejs
          node --version
          npm --version
        '''
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
        // Fixed credentialsId to match environment variable
        withCredentials([usernamePassword(
          credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",  // Use the environment variable
          passwordVariable: 'DOCKERHUB_PASSWORD',
          usernameVariable: 'DOCKERHUB_USERNAME'
        )]) {
          sh 'docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}'
          sh 'docker push ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER}'
        }
      }
    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}
