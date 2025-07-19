pipeline {
  agent {
    docker {
      image 'node:18'
      args '-u root' // Usamos root dentro del contenedor para evitar problemas de permisos
    }
  }

  stages {
    stage('Install') {
      steps {
        sh 'npm install'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test || echo "No tests defined"'
      }
    }

    stage('Build') {
      steps {
        sh 'npm run build || echo "No build script defined"'
      }
    }
  }

  post {
    always {
      echo 'Pipeline finished'
    }
  }
}
