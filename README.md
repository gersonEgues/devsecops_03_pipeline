**Guía Práctica: Jenkins + Docker + Node.js + GitHub (CI con Pipeline y Token)**

---

### 🌟 Objetivo

Automatizar el proceso de integración continua (CI) usando Jenkins, Docker y un proyecto Node.js alojado en GitHub privado, utilizando un `Jenkinsfile` y token de autenticación.

---

### ✅ Requisitos previos

- Tener instalado Docker
- Tener una cuenta GitHub y un repo privado (ej: `https://github.com/gersonEgues/devsecops_01_pipeline`)
- Tener un GitHub **Personal Access Token (PAT)** válido

---

### 🚀 Paso 1: Levantar Jenkins con Docker

1. Crear una carpeta local (ej: `jenkins-docker`)
2. Dentro, crear un archivo `docker-compose.yml` con el siguiente contenido:

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

3. Levantar Jenkins:

```bash
docker compose up -d
```

4. Acceder a Jenkins: [http://localhost:8080](http://localhost:8080)

5. Copiar la clave de desbloqueo inicial desde el contenedor:

```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

6. Completar instalación recomendada y crear tu usuario.

---

### 🔧 Paso 2: Instalar plugins necesarios

Desde "Manage Jenkins" > "Manage Plugins":

- Pipeline
- Git
- GitHub Integration
- NodeJS

---

### 🎨 Paso 3: Configurar herramienta Node.js

Desde "Manage Jenkins" > "Global Tool Configuration":

- Sección NodeJS > Add NodeJS > Nombre: `Node 18`
- Marcar: ☑ Install automatically > Versión estable (ej. 18.x)

---

### 🔐 Paso 4: Crear credenciales para GitHub

Desde "Manage Jenkins" > "Credentials" > Global:

- Tipo: **Username with password**
  - Username: tu usuario GitHub
  - Password: tu Personal Access Token (PAT)

Guarda con ID: `github-token`

---

### 🧱 Paso 5: Crear Pipeline Job

1. Ir a Jenkins > Nuevo Item
2. Nombre: `node-pipeline-job`
3. Tipo: `Pipeline`

#### Configura:

- En **Pipeline** > Definition: `Pipeline script from SCM`
  - SCM: `Git`
  - URL: `https://github.com/gersonEgues/devsecops_01_pipeline.git`
  - Credentials: selecciona `github-token`
  - Branch: `*/main`
  - Script Path: `Jenkinsfile` (asegúrate que el archivo esté en la raíz del repo)

---

### 📝 Ejemplo de Jenkinsfile

```groovy
pipeline {
  agent any

  tools {
    nodejs 'Node 18'
  }

  stages {
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
```

---

### ⏲️ Paso 6: Configurar Poll SCM

En tu Pipeline Job > Configuración:

- Habilitar: ☑ Poll SCM
- Schedule: `* * * * *` (Jenkins revisa cada minuto si hay cambios)

---

### 🧪 Resultado esperado

Al hacer `git push` al repo:

- Jenkins detectará el cambio al minuto
- Ejecutará los stages definidos
- Verás en Console Output algo como:

```
[Pipeline] Start of Pipeline
npm install
npm test
Test ejecutado correctamente
[Pipeline] End of Pipeline
```

---

### ✅ Revisión final

- Jenkinsfile debe estar en el root del repo
- Jenkins debe tener acceso al repo privado vía token
- Poll SCM activado cada minuto
- Los stages deben ejecutarse correctamente

---

🚀 ¡Felicidades! Ya tienes tu CI básico con Jenkins + GitHub privado + Pipeline funcionando 🔥
