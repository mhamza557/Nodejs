pipeline {
  agent any

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'
    CI = 'true'
    NODE_HOME = "${WORKSPACE}/.nodejs"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup Environment') {
      steps {
        script {
          // Download the .tar.gz version which doesn't require xz
          sh '''
            mkdir -p ${NODE_HOME}
            curl -fsSL https://nodejs.org/dist/v18.20.0/node-v18.20.0-linux-x64.tar.gz -o node.tar.gz
            tar -xzf node.tar.gz -C ${NODE_HOME} --strip-components=1
            export PATH="${NODE_HOME}/bin:${PATH}"
            node --version
            npm --version
          '''
        }
      }
    }

    stage('Install dependencies') {
      steps {
        sh '''
          export PATH="${NODE_HOME}/bin:${PATH}"
          npm ci --prefer-offline --audit false
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          export PATH="${NODE_HOME}/bin:${PATH}"
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
