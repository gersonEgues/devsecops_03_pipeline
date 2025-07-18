**Guía Práctica: Jenkins + Docker + Node.js + GitHub (CI Básico)**

---

### 🌟 Objetivo

Automatizar el proceso de integración continua (CI) usando Jenkins, Docker y un proyecto Node.js alojado en GitHub.

---

### ✅ Requisitos previos

- Tener instalado Docker
- Proyecto público en GitHub (ej: `https://github.com/gersonEgues/devsecops_01_pipeline`)

---

### 🚀 Paso 1: Levantar Jenkins con Docker

1. Crear el volumen de manera manual (ya que en la configuracion de docker compose, le decimos que use un volumen externo `external: true` )

```yaml
docker volume create jenkins_home
```

2. Verificar volumen creado

```yaml
docker volume ls 
```

3. verificar el volumen creado: 
4. Crear una carpeta local (ej: `jenkins-docker`)
5. Dentro, crear un archivo `docker-compose.yml` con el siguiente contenido:

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
    external: true
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
7. Seleccionar la opción `Select plugins to install`e instalar los plugions necesarios como: `Docker, NOde, Git, Github, Gitlab`
---

### 🔧 Paso 2: Instalar plugins necesarios

Desde "Manage Jenkins" > "Manage Plugins":

- NodeJS
- Git
- GitHub

---

### 🎨 Paso 3: Configurar herramienta Node.js

Desde "Manage Jenkins" > "Global Tool Configuration":

- Sección NodeJS > Add NodeJS > Nombre: `Node 18`
- Marcar: ☑ Install automatically > Versión estable (ej. 18.x)

---

### 📂 Paso 4: Crear el Job Freestyle

1. En Jenkins, haz clic en "Nuevo Item"

2. Nombre: `node-ci-job`

3. Tipo: `Freestyle project`

4. En "Source Code Management":

   - Git
   - URL: `https://github.com/gersonEgues/devsecops_01_pipeline.git`
   - Branch: `*/main`

5. En "Build Triggers":

   - ☑ Poll SCM: `* * * * *` (cada minuto)

6. En "Build Environment":

   - ☑ Provide Node & npm bin/folder to PATH > seleccionar `Node 18`

7. En "Build":

   - Add build step > Execute shell:

```bash
npm install
npm test
```

8. Guardar el Job

---

### 🏋️ Verificar ejecuciones

- Jenkins clonará el repo en: `/var/jenkins_home/workspace/node-ci-job`

- Ver resultado en "Console Output" del build

- Validar salida del test: `Test ejecutado correctamente`

- Ingresar al contenedor jenkins: `docker exec -it jenkins bash`

- Ingresar a la ruta: `/var/jenkins_home/workspace/node-ci-job`, ver los archivos copiados

- Esta cnfigurado de tal manera que revisa el repositorio cada minuto si hay cambios, si es asi, copia el contendido de el repositorio al contenedor de jenkins

- para este ejemplo no se uso tnkens porque el respositorio es publico
---

### ⚠️ Errores comunes

- **onsole.log**: Typo (debe ser `console.log`)
- **package.json not found**: repo mal clonado o vacío
- **Git no instalado**: entrar al contenedor y hacer:
  ```bash
  apt-get update && apt-get install -y git
  ```

---

### 🔄 Para volver a hacerlo rápido

1. Clonar tu repo GitHub (si no lo tienes local)
2. Crear `jenkins-docker/` con `docker-compose.yml`
3. `docker compose up -d`
4. Ir a [localhost:8080](http://localhost:8080)
5. Crear Job freestyle con los pasos anteriores
6. Empujar cambios a GitHub y ver Jenkins ejecutándose

---

### 🚧 Opcional: siguiente nivel

- Usar Webhook de GitHub para builds instantáneos
- Migrar a Jenkinsfile declarativo
- Integrar `jest` o `mocha` para tests reales
- Construir imagen Docker del proyecto
- Desplegar automáticamente

---

🚀 ¡Felicitaciones! Ya dominaste el flujo básico de CI con Jenkins y Node.js
