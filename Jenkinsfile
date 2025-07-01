pipeline {
  agent any

  environment {
    DOCKERHUB_REGISTRY = 'nocnex/nodejs-app-v2'
    DOCKERHUB_CREDENTIALS_ID = 'dockerhublogin'
    CI = 'true'
    NODE_HOME = "${WORKSPACE}/.nodejs"
    PATH = "${NODE_HOME}/bin:${PATH}"
    KUBE_CONFIG = credentials('kubeconfig') // Store kubeconfig as Jenkins credential
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
          sh """
            mkdir -p ${NODE_HOME}
            curl -fsSL https://nodejs.org/dist/v18.20.0/node-v18.20.0-linux-x64.tar.gz -o ${WORKSPACE}/node.tar.gz
            tar -xzf ${WORKSPACE}/node.tar.gz -C ${NODE_HOME} --strip-components=1
            rm -f ${WORKSPACE}/node.tar.gz
            node --version
            npm --version
          """
        }
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm ci --prefer-offline --audit false'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
    }

    stage('Build and Push Docker Image') {
      when {
        expression { sh(returnStatus: true, script: 'command -v docker') == 0 }
      }
      steps {
        script {
          withCredentials([usernamePassword(
            credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
            passwordVariable: 'DOCKERHUB_PASSWORD',
            usernameVariable: 'DOCKERHUB_USERNAME'
          )]) {
            sh """
              docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}
              docker build -t ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER} .
              docker push ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER}
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      when {
        expression { sh(returnStatus: true, script: 'command -v kubectl') == 0 }
      }
      steps {
        script {
          // Write kubeconfig to file
          writeFile file: "${WORKSPACE}/kubeconfig", text: "${KUBE_CONFIG}"
          
          sh """
            export KUBECONFIG=${WORKSPACE}/kubeconfig
            kubectl apply -f kubernetes/
            kubectl set image deployment/nodejs-app nodejs-container=${DOCKERHUB_REGISTRY}:${BUILD_NUMBER} --record
            kubectl rollout status deployment/nodejs-app
          """
        }
      }
    }
  }

  post {
    always {
      sh 'command -v docker >/dev/null 2>&1 && docker logout || true'
      cleanWs()
    }
    success {
      echo 'Deployment completed successfully!'
    }
    failure {
      echo 'Deployment failed'
    }
  }
}
