pipeline {
  agent any

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'
    CI = 'true'
    NODE_ENV = 'production'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup Node.js') {
      steps {
        script {
          // Install Node.js 18 directly on the agent
          sh '''
            curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
            apt-get install -y nodejs
            node --version
            npm --version
          '''
        }
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
      when {
        expression { sh(returnStatus: true, script: 'command -v docker') == 0 }
      }
      steps {
        script {
          sh 'docker build -t ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER} .'
        }
      }
    }

    stage('Push Docker image') {
      when {
        expression { sh(returnStatus: true, script: 'command -v docker') == 0 }
      }
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
        sh 'command -v docker >/dev/null 2>&1 && docker logout || true'
        cleanWs()
      }
    }
  }
}
