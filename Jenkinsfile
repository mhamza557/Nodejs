pipeline {
  agent {
    docker {
      image 'node:18-alpine'
      args '--user root'
      reuseNode true
    }
  }

  tools {
    nodejs 'NodeJS-18' // Using Jenkins NodeJS plugin (configure in Global Tool Configuration)
  }

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
        sh 'npm cache clean --force' // Clean up npm cache
      }
    }

    stage('Lint & Test') {
      steps {
        sh 'npm run lint || true' // Optional lint step, continues if fails
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
              -t ${DOCKERHUB_REGISTRY}:latest \
              --push .
          """
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        script {
          // Simple verification that image was pushed
          sh """
            docker pull ${DOCKERHUB_REGISTRY}:${BUILD_NUMBER} && \
            echo "Successfully verified image exists in registry"
          """
        }
      }
    }
  }

  post {
    always {
      script {
        sh 'docker logout || true'
        cleanWs()
      }
    }
    success {
      echo 'Pipeline completed successfully'
      archiveArtifacts artifacts: '**/build/**/*' // Optional: archive build artifacts
    }
    failure {
      echo 'Pipeline failed'
      // Add any failure notifications here if needed
    }
  }
}
