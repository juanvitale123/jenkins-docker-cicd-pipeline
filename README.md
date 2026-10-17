# CI/CD Pipeline con Jenkins, Docker, SonarQube y AWS ECR

Pipeline declarativo de Jenkins que automatiza el ciclo completo de una aplicación Java: compilación, tests, análisis de calidad con SonarQube, construcción de la imagen Docker y publicación en un registro privado de AWS ECR.

Como aplicación de ejemplo se usa el proyecto Java de código abierto **vprofile** (`hkhcoder/vprofile-project`, rama `docker`). El trabajo de este repositorio es el pipeline y la infraestructura de CI/CD, no la aplicación.

## Flujo del pipeline

```
 GitHub ─► Build (Maven) ─► Unit tests ─► Checkstyle ─► SonarQube ─► Quality Gate ─► Docker build ─► Push a ECR
                │                                                       │
         archiva el .war                                  aborta si no se cumple
```

| Etapa | Qué hace |
|---|---|
| Validate parameters | Comprueba que se ha indicado la cuenta de AWS |
| Fetch code | Clona la aplicación desde GitHub |
| Build | `mvn install` y archiva el `.war` generado |
| Unit tests | `mvn test` |
| Checkstyle analysis | Análisis de estilo del código |
| SonarQube analysis | Envía código, tests, cobertura y Checkstyle a SonarQube |
| Quality Gate | Detiene el pipeline si no se supera el umbral de calidad |
| Build app image | Construye la imagen con un Dockerfile multietapa |
| Push image to ECR | Sube la imagen con la etiqueta del build y `latest` |
| Limpieza (post) | Borra solo las imágenes creadas por este job |

## Tecnologías

- Jenkins (pipeline declarativo)
- Maven y JDK 17
- SonarQube + Quality Gates
- Docker
- AWS ECR (credenciales gestionadas con el plugin Amazon ECR)
- Git

## Requisitos en Jenkins

| Elemento | Nombre configurado |
|---|---|
| Herramienta Maven | `mavenpractice` |
| Herramienta JDK | `JDK17` |
| Scanner de SonarQube | `sonar6.2` |
| Servidor SonarQube | `sonarserver` (con webhook a Jenkins para el Quality Gate) |
| Credenciales AWS | `awscreds` |
| Plugins | Pipeline, Docker Pipeline, Amazon ECR, SonarQube Scanner |

## Uso

1. Crear un job de tipo *Pipeline* apuntando a este repositorio (`Jenkinsfile`).
2. Ejecutar *Build with Parameters* indicando `AWS_ACCOUNT_ID` y, si hace falta, `AWS_REGION`.

## Decisiones de diseño

- **ID de cuenta como parámetro**: no queda escrito en el código.
- **Quality Gate bloqueante**: el código que no cumple los mínimos no llega a la imagen.
- **Limpieza selectiva**: borrar todas las imágenes del agente (`docker rmi $(docker images -q)`) afectaría a otros jobs; solo se eliminan las de este build.

## Estructura

```
.
├── Jenkinsfile
└── README.md
```
