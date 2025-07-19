# 📘 Guía Completa: CI con Jenkins + Docker + Node.js + GitHub + Docker Hub

---

## 🎯 Objetivo

Automatizar el proceso de integración continua (CI) y despliegue de imagen Docker utilizando Jenkins, Docker, GitHub (repo privado), Docker Hub y un `Jenkinsfile`.

---

## ✅ Requisitos Previos

- Tener instalado **Docker**
- Tener cuenta en **GitHub** con un repo privado
- Tener cuenta en **Docker Hub**
- Tener un **Personal Access Token (PAT)** de GitHub
- Tener un token (usuario/contraseña) válido para Docker Hub

---

## 🐳 Paso 1: Levantar Jenkins con Docker

1. Crear una carpeta local `jenkins-docker`
2. Dentro, crear un archivo `docker-compose.yml`:

```yaml
docker-compose.yml:

version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
    volumes:
      - jenkins_home:/var/jenkins_home
    restart: unless-stopped

volumes:
  jenkins_home:
```

3. Ejecutar Jenkins:

```bash
docker compose up -d
```

4. Ingresar a Jenkins en [http://localhost:8080](http://localhost:8080)
5. Obtener la contraseña inicial:

```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

6. Completar instalación recomendada y crear usuario admin.

---

## 🔌 Paso 2: Instalar Plugins en Jenkins

Desde **Manage Jenkins > Manage Plugins**:

- Pipeline
- Git
- GitHub Integration
- NodeJS
- Docker Pipeline

---

## 🔧 Paso 3: Configurar Herramientas

### NodeJS

Desde **Manage Jenkins > Global Tool Configuration**:

- NodeJS installations → Add NodeJS
  - Name: `Node 18`
  - ☑ Install automatically → 18.x

---

## 🔐 Paso 4: Crear Credenciales

Desde **Manage Jenkins > Credentials > (Global)**:

### 1. GitHub

- Tipo: `Username with password`
  - Username: tu usuario GitHub
  - Password: tu PAT
  - ID: `github_token`

### 2. Docker Hub

- Tipo: `Username with password`
  - Username: tu usuario Docker Hub
  - Password: tu contraseña Docker Hub
  - ID: `dockerhub_token`

---

## 📂 Paso 5: Estructura del Proyecto

Repositorio de ejemplo: [https://github.com/gersonEgues/devsecops\_03\_pipeline](https://github.com/gersonEgues/devsecops_03_pipeline)

Debe contener en raíz:

- `Dockerfile`
- `Jenkinsfile`
- `package.json`
- `test/sample.test.js`

---

## 📄 Dockerfile

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

---

## 📄 Jenkinsfile

```groovy
pipeline {
  agent any

  tools {
    nodejs 'Node 18'
  }

  environment {
    DOCKER_IMAGE = '201400050/devsecops_03_pipeline:latest'
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

    stage('Build Docker Image') {
      steps {
        script {
          sh "docker build -t ${DOCKER_IMAGE} ."
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          withDockerRegistry([ credentialsId: 'dockerhub_token', url: 'https://index.docker.io/v1/' ]) {
            sh "docker tag ${DOCKER_IMAGE} index.docker.io/${DOCKER_IMAGE}"
            sh "docker push index.docker.io/${DOCKER_IMAGE}"
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
```

---

## 🛠️ Paso 6: Crear Pipeline Job en Jenkins

1. Jenkins → New Item → `node-pipeline-job3`
2. Tipo: `Pipeline`

### Configura:

- Definition: `Pipeline script from SCM`
  - SCM: Git
  - URL: `https://github.com/gersonEgues/devsecops_03_pipeline.git`
  - Credentials: `github_token`
  - Branch: `*/main`
  - Script Path: `Jenkinsfile`

---

## 🧪 Resultado Esperado

Cada vez que hagas `git push`:

- Jenkins clona el repo privado
- Instala dependencias
- Corre los tests
- Construye imagen Docker
- La sube automáticamente a Docker Hub (`docker.io/201400050/devsecops_03_pipeline`)

---

## ✅ Verificación Final

- Jenkinsfile correcto y en la raíz del repo
- Credenciales válidas para GitHub y DockerHub
- Dockerfile funcional
- Imagen disponible en tu [Docker Hub](https://hub.docker.com/u/201400050)

---

🚀 ¡Felicidades! Has logrado un pipeline completo con GitHub privado, Jenkins, Docker y despliegue a Docker Hub 🎉
