[authapp-ci-README.md](https://github.com/user-attachments/files/27141187/authapp-ci-README.md)
<div align="center">

# 🔐 AuthApp CI

### Pipeline CI con Jenkins · Java 21 · Maven · SonarQube · Docker

![Java](https://img.shields.io/badge/Java_21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)
![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?style=for-the-badge&logo=sonarqube&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

</div>

---

## 📌 ¿Qué hace este proyecto?

Pipeline de **Integración Continua (CI)** para una aplicación Java de autenticación. Con cada cambio en el repositorio, Jenkins compila el proyecto con Maven, ejecuta las pruebas unitarias y envía los resultados a SonarQube para análisis estático de calidad de código. Todo el entorno corre en contenedores Docker.

---

## 🏗️ Arquitectura

```
GitHub (push) ──▶ Jenkins ──▶ Maven (build + test) ──▶ SonarQube (análisis)
                     │                                         │
                     └── Reporte de éxito/fallo ◀─────────────┘
```

**Servicios Docker:**

```
docker-compose.yml
├── jenkins       → :8080  (CI server)
└── sonarqube     → :9000  (análisis de calidad)
```

---

## 🗂️ Estructura del proyecto

```
authapp-ci/
├── authapp/              # Código fuente Java
│   ├── src/
│   │   ├── main/java/    # Lógica de autenticación
│   │   └── test/java/    # Pruebas unitarias
│   └── pom.xml
├── jenkins/
│   └── Dockerfile        # Imagen Jenkins personalizada con Java + Maven
├── docs/                 # Documentación de configuración
├── Jenkinsfile           # Definición del pipeline CI
└── docker-compose.yml    # Orquestación de servicios
```

---

## ⚙️ Requisitos

- Docker y Docker Compose instalados
- Git
- Puertos libres: `8080` (Jenkins), `9000` (SonarQube)

---

## 🚀 Levantar el entorno

```bash
# 1. Clonar el repositorio
git clone https://github.com/LujoMontero/authapp-ci.git
cd authapp-ci

# 2. Construir y levantar los contenedores
docker compose up --build -d

# 3. Obtener la contraseña inicial de Jenkins
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 4. Acceder a Jenkins
open http://localhost:8080

# 5. Acceder a SonarQube
open http://localhost:9000
```

---

## 🔧 Configuración de Jenkins

Una vez dentro de Jenkins en `http://localhost:8080`:

1. Ingresar la contraseña inicial obtenida en el paso anterior
2. Instalar los **plugins sugeridos**
3. Configurar herramientas en **Manage Jenkins → Tools**:
   - **JDK 21**: `readlink -f $(which java)` para obtener la ruta
   - **Maven 3.x**: `/usr/share/maven`
   - **SonarQube**: URL `http://sonarqube:9000`
4. Crear nuevo **Pipeline Job**:
   - Origen: Git → `https://github.com/LujoMontero/authapp-ci.git`
   - Branch: `main`
   - Script Path: `Jenkinsfile`

---

## 📄 Jenkinsfile — Pipeline completo

```groovy
pipeline {
  agent any

  tools {
    jdk 'Java21'
    maven 'Maven3'
  }

  environment {
    SONARQUBE_ENV = 'SonarQube'
  }

  stages {
    stage('Build y Test') {
      steps {
        dir('authapp') {
          sh 'mvn clean test'
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE_ENV}") {
          dir('authapp') {
            sh '''
              mvn sonar:sonar \
                -Dsonar.projectKey=authapp \
                -Dsonar.host.url=http://sonarqube:9000 \
                -Dsonar.login=$SONAR_TOKEN
            '''
          }
        }
      }
    }
  }

  post {
    success { echo '✅ Pipeline completado correctamente' }
    failure { echo '❌ Pipeline falló — revisar logs' }
  }
}
```

---

## 📊 Análisis de calidad con SonarQube

Generar token en SonarQube antes de ejecutar el pipeline:

```
http://localhost:9000 → My Account → Security → Generate Tokens
```

Agrega el token como **credential** en Jenkins:

```
Manage Jenkins → Credentials → Global → Add → Secret text
ID: SONAR_TOKEN
```

**Métricas que analiza SonarQube:**

| Categoría | Qué detecta |
|---|---|
| Bugs | Errores de lógica y NPE potenciales |
| Vulnerabilidades | Problemas de seguridad en el código |
| Code Smells | Código difícil de mantener |
| Cobertura | % de código cubierto por pruebas |
| Duplicaciones | Código repetido |

---

## 🧪 Ejecutar pruebas manualmente

```bash
cd authapp
mvn clean test

# Ver reporte de pruebas
open target/surefire-reports/index.html
```

---

## 🛑 Detener el entorno

```bash
# Detener servicios
docker compose down

# Detener y limpiar volúmenes (borra datos Jenkins y SonarQube)
docker compose down -v --remove-orphans
```

---

## 🗺️ Roadmap

- [x] Pipeline CI con Jenkins + Maven
- [x] Análisis estático con SonarQube
- [ ] Notificaciones a Slack en fallo/éxito
- [ ] Despliegue automático (CD) a servidor de staging
- [ ] Quality Gate que bloquee merge si hay bugs críticos

---

## 💡 Conceptos aplicados

- **CI (Integración Continua)**: compilación y pruebas automáticas en cada commit
- **Análisis estático**: SonarQube detecta problemas sin ejecutar el código
- **Pipeline as Code**: el `Jenkinsfile` vive en el repositorio junto al código
- **Infraestructura como contenedores**: entorno reproducible con Docker Compose

---

## 👨‍💻 Autor

**Luis Montero** · [GitHub](https://github.com/LujoMontero) · [LinkedIn](https://www.linkedin.com/in/luis-montero-if/)
