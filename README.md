## Fabio Murcia

Estudiante de último año de Ingeniería de Sistemas y Computación en la Universidad
Nacional de Colombia, e ingeniero de datos y nube.

Trabajo en backend distribuido, pipelines de datos y plataformas web: microservicios,
mensajería asíncrona, bases de datos e infraestructura como código.

## Dónde está mi código

- **[recipy-swarch](https://github.com/recipy-swarch)** — plataforma web de recetas en microservicios
- **[TeamC-swarchqua](https://github.com/TeamC-swarchqua)** — billetera digital en AWS y trabajo de calidad de software

Los dos primeros proyectos que siguen son de curso, desarrollados en equipos de siete
personas.

## Proyectos

### [CocoCash](https://github.com/TeamC-swarchqua/cococash)

Billetera digital: transferencias entre cuentas y generación de extractos, con
arquitectura orientada a eventos sobre AWS. El repositorio contiene tanto el código de
los servicios como la infraestructura.

Escribí de punta a punta el servicio de billetera (`cococash-wallet-ms`, TypeScript y
Express sobre PostgreSQL), el servicio de transacciones (`cococash-transaction-ms`, Go
con DynamoDB) y tres funciones Lambda en Go (`pdf-maker`, `get-accounts`,
`link-generator`). También el middleware de autenticación con Cognito (JWT + JITP), el
endpoint de depósito con bloqueo a nivel de fila, y parte del Terraform de red, ECS
Fargate y API Gateway.

`TypeScript` · `Go` · `Terraform` · `AWS (ECS Fargate, Lambda, RDS, DynamoDB, SNS, SQS, Cognito, ALB, VPC)` · `Docker`

### [Recipy](https://github.com/recipy-swarch/recipy)

Red social de recetas construida como conjunto de microservicios, con despliegue en
Docker Compose y Kubernetes.

Fui responsable del microservicio de recetas (`recipe-ms`): API en FastAPI con GraphQL
sobre MongoDB, validación de JWT, y los endpoints de recetas, comentarios y likes.
Escribí también el servicio de caché sobre Redis (`recipy-cache`) y su integración con
`recipe-ms`. Del lado del cliente toqué la capa de servicios del frontend en Next.js y
el enrutamiento del API Gateway en NestJS, además del `docker-compose` y los scripts de
inicialización de la base de datos.

`Python (FastAPI, Strawberry GraphQL)` · `MongoDB` · `Redis` · `TypeScript (Next.js, NestJS)` · `Docker`

### [Reto de automatización de entorno](https://github.com/GorenNM/devops-reto-automatizacion-entorno)

Repositorio propio y pequeño: una aplicación Node.js contenerizada, un módulo de
Terraform que construye y publica la imagen, y un script de gestión del entorno Docker.
Sirve como muestra de infraestructura como código escrita íntegramente por mí.

`Terraform` · `Docker` · `Node.js` · `Bash`

### [gopdfsuit-qua](https://github.com/TeamC-swarchqua/gopdfsuit-qua)

**Fork** del proyecto [GoPdfSuit](https://github.com/chinmay-sawant/gopdfsuit) (MIT), usado
como base para un trabajo de calidad de software. El código del generador de PDF no es
mío.

Mi aporte encima del fork: la suite de pruebas de integración y end-to-end (Playwright
para el frontend, pruebas en Go para el backend), el reemplazo de la autenticación con
Google OAuth por un servicio propio en el middleware, y el análisis de seguridad y
mantenibilidad del proyecto (ZAP, Trivy, gosec, golangci-lint, modelo de amenazas).

`Go` · `Playwright` · `Vitest` · `Análisis estático y DAST`

## Stack

- **Lenguajes** — Python, Go, TypeScript / JavaScript, SQL, Bash
- **Nube** — AWS: ECS Fargate, Lambda, RDS, DynamoDB, SNS, SQS, Cognito, API Gateway, ALB, VPC, CloudWatch, ECR
- **Datos** — PostgreSQL, MongoDB, DynamoDB, Redis
- **Infraestructura** — Terraform, Docker, Docker Compose, Kubernetes, Nginx
- **Backend** — FastAPI, GraphQL, Express, NestJS, arquitecturas orientadas a eventos
- **Frontend** — Next.js, React

## Contacto

- femurcia6@gmail.com
- [linkedin.com/in/fabio-murcia](https://linkedin.com/in/fabio-murcia)
