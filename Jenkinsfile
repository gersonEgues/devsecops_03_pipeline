pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-credentials-id' // Cambia por el ID que pusiste en Jenkins para Docker Hub
    IMAGE_NAME = '201400050/devsecops_03_pipeline'
    IMAGE_TAG = "latest"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build App') {
      agent {
        docker {
          image 'node:18'
          args '-u root'
        }
      }
      steps {
        sh 'npm install'
        sh 'npm test || echo "No tests defined"'
        sh 'npm run build || echo "No build script defined"'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
            dockerImage.push()
          }
        }
      }
    }
  }

  post {
    always {
      echo 'Pipeline finished'
    }
  }
}
