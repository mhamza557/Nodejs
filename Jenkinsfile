pipeline {
  agent any

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'
    CI = 'true'
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
          // Alternative Node.js installation without requiring root
          sh '''
            mkdir -p ${WORKSPACE}/.nodejs
            curl -fsSL https://nodejs.org/dist/v18.20.0/node-v18.20.0-linux-x64.tar.xz -o node.tar.xz
            tar -xJf node.tar.xz -C ${WORKSPACE}/.nodejs --strip-components=1
            export PATH="${WORKSPACE}/.nodejs/bin:${PATH}"
            node --version
            npm --version
          '''
        }
      }
    }

    stage('Install dependencies') {
      steps {
        sh '''
          export PATH="${WORKSPACE}/.nodejs/bin:${PATH}"
          npm ci --prefer-offline --audit false
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          export PATH="${WORKSPACE}/.nodejs/bin:${PATH}"
          npm test
        '''
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
  }

  post {
    always {
      sh 'command -v docker >/dev/null 2>&1 && docker logout || true'
      cleanWs()
    }
  }
}
