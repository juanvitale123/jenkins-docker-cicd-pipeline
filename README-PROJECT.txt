
markdown
Copiar
Editar
# CI/CD Pipeline con Jenkins, Docker y SonarQube

Este proyecto implementa un pipeline completo de integración y entrega continua (CI/CD) utilizando Jenkins, Docker, SonarQube y Nexus.

## 🔧 Tecnologías usadas

- Jenkins
- SonarQube
- Nexus Repository
- Docker
- Maven
- Git

## 🚀 ¿Qué hace el pipeline?

1. Clona el repositorio de una app Java desde GitHub.
2. Compila con Maven y genera el `.war` (sin tests).
3. Ejecuta tests unitarios y análisis de calidad con SonarQube.
4. Sube artefactos al repositorio Nexus.
5. Construye una imagen Docker de la app.
6. Empuja la imagen al registro privado en AWS ECR.
7. Limpia recursos del entorno local.

## 📁 Estructura

```bash
.
├── Jenkinsfile
├── Docker-files/
│   └── app/
│       └── multistage/
│           └── Dockerfile
├── README.md