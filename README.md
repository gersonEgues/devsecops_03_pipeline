
# Manual Completo para Configurar Jenkins con Docker y Pipeline para Repositorio Privado en GitHub

## 1. Requisitos Previos
- Tener instalado Docker y Docker Compose en tu máquina host.
- Acceso a un repositorio privado en GitHub con un `Jenkinsfile` en la raíz.
- Tener un token de acceso personal (PAT) de GitHub para acceder al repositorio privado.

## 2. Configuración de Docker Compose para Jenkins

Archivo `docker-compose.yml` básico para levantar Jenkins con acceso al socket Docker del host:

```yaml
version: '3'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always

volumes:
  jenkins_home:
    external: true
```

## 3. Crear volumen externo para Jenkins

```bash
docker volume create jenkins_home
```

Luego levantar Jenkins con:

```bash
docker-compose up -d
```

## 4. Acceder a Jenkins

Abrir navegador en `http://localhost:8080` (o la IP/puerto que uses).

## 5. Instalar plugins necesarios

En Jenkins, ir a `Manage Jenkins` > `Manage Plugins` > pestaña `Available` y buscar e instalar:

- **Docker Pipeline** (para usar contenedores docker dentro de pipelines)
- **GitHub Branch Source** (para integración con GitHub)
- **Git plugin** (manejo de Git en Jenkins)
- Otros plugins necesarios para tu pipeline si es requerido.

Reiniciar Jenkins si es necesario.

## 6. Configurar permisos de Docker en el contenedor Jenkins

Para que Jenkins pueda usar Docker dentro del contenedor:

- Montar el socket docker del host dentro del contenedor (ya está en el `docker-compose.yml`).
- Asegurarse que el grupo `docker` tiene acceso a `/var/run/docker.sock`.
- En el contenedor Jenkins, agregar el usuario `jenkins` al grupo `docker`:

```bash
docker exec -it jenkins bash
groupadd docker # si no existe
usermod -aG docker jenkins
exit
```

Reiniciar el contenedor Jenkins para que tome los nuevos permisos.

## 7. Configurar credenciales en Jenkins para GitHub

- En Jenkins, ir a `Manage Jenkins` > `Manage Credentials` > `Global` > `Add Credentials`
- Tipo: `Secret text`
- Pegar tu token personal de GitHub (PAT) con permisos de lectura al repositorio privado.
- Guardar con ID (ejemplo: `github_token`)

## 8. Crear Pipeline en Jenkins

- Crear un nuevo Job > Pipeline
- En `Pipeline` seleccionar `Pipeline script from SCM`
- Tipo SCM: `Git`
- URL: tu repo privado, por ejemplo: `https://github.com/tu_usuario/tu_repo.git`
- Credentials: seleccionar el token `github_token` creado antes
- Branches to build: `main` o la rama que uses
- Script Path: `Jenkinsfile` (asumiendo que está en la raíz del repo)

## 9. Jenkinsfile ejemplo para pipeline con contenedor Node.js

```groovy
pipeline {
  agent {
    docker {
      image 'node:18'
      args '-u root'
    }
  }
  environment {
    // Variables de entorno si es necesario
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
```

## 10. Prueba y ejecución

- Haz commit y push de tu código y Jenkinsfile al repositorio.
- Jenkins detectará cambios si usas polling o webhook y ejecutará el pipeline.
- Puedes revisar logs y estado en la interfaz de Jenkins.

## 11. Acceder al workspace de Jenkins

Para revisar los archivos que Jenkins clonó, puedes acceder al contenedor y navegar:

```bash
docker exec -it jenkins bash
cd /var/jenkins_home/workspace/nombre_del_job
ls -la
```

## 12. Consejos finales

- Siempre verifica que el contenedor Jenkins pueda ejecutar comandos Docker (usa el socket montado y permisos).
- Usa tokens personales de GitHub para repositorios privados.
- Si tienes errores de permiso, revisa grupos y permisos del socket Docker.
- Mantén actualizado Jenkins y plugins para evitar incompatibilidades.

---

Este manual cubre la instalación desde cero, configuración, ejecución y solución básica de problemas para pipelines con Jenkins, Docker y repositorios privados en GitHub.
