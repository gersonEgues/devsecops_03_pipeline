pipeline {
  agent {
    docker {
      image 'node:24'
      args '-u root'
    }
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    
    stage('Install') {
      steps {
        sh 'npm install'
      }
    }
    
    stage('Test') {
      steps {
        sh 'npm test'
      }
    }
  }
}
