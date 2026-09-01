# Hola, soy Fabio 👋

Estudiante de último año de **Ingeniería de Sistemas y Computación** en la Universidad
Nacional de Colombia, e ingeniero de datos y nube.

Construyo backend distribuido, pipelines de datos y plataformas web. Me gusta la parte
que no se ve: la cola que no pierde mensajes, la transacción que no se corrompe cuando
dos usuarios tocan la misma cuenta al mismo tiempo, el `terraform apply` que levanta
todo desde cero.

> ** Mi código vive en las organizaciones de los
> equipos con los que trabajo:
>
> 🍳 **[recipy-swarch](https://github.com/recipy-swarch)** · 🥥 **[TeamC-swarchqua](https://github.com/TeamC-swarchqua)**

Los dos proyectos grandes son de curso, hechos en equipos de siete personas. En los
diagramas marqué **en verde lo que escribí yo**.

---

## 🥥 [CocoCash](https://github.com/TeamC-swarchqua/cococash) — billetera digital en AWS

Transferencias entre cuentas y generación de extractos bancarios, con arquitectura
orientada a eventos. El repositorio tiene el código de los servicios **y** la
infraestructura completa en Terraform.

```mermaid
flowchart LR
    U([Usuario]) --> AG[API Gateway]
    AG --> W["cococash-wallet-ms<br/>TypeScript · Express"]
    AG --> L["link-generator<br/>Go · Lambda"]
    W <--> R[("PostgreSQL<br/>RDS")]
    W -->|"transfer.initiated<br/>deposit.completed"| SNS((SNS))
    SNS --> SQS[SQS]
    SQS --> T["cococash-transaction-ms<br/>Go"]
    T <--> D[(DynamoDB)]
    EB[EventBridge] --> GA["get-accounts<br/>Go · Lambda"]
    GA --> SNS
    T --> P["pdf-maker<br/>Go · Lambda"]
    P --> S3[(S3)]
    L --> S3

    classDef mio fill:#238636,stroke:#2ea043,stroke-width:2px,color:#fff
    classDef equipo fill:#484f58,stroke:#6e7681,color:#fff
    class W,T,L,GA,P mio
    class AG,EB equipo
```

**Lo que he trabajado:**

| Componente | Qué hace | Mío |
|---|---|---|
| `cococash-wallet-ms` | Cuentas, transferencias y depósitos sobre PostgreSQL  |
| `cococash-transaction-ms` | Consume eventos y construye el libro mayor en DynamoDB |
| `cococash-pdf-maker` | Genera el extracto en PDF y lo sube a S3  |
| `cococash-get-accounts` | Dispara la generación mensual desde EventBridge  |
| `cococash-link-generator` | Devuelve una URL prefirmada de S3 para descargar el extracto  |
| Terraform de red, ECS y API Gateway | VPC, ALB, Fargate, IAM  |

También el middleware de autenticación con **Cognito** (JWT + JITP) y el endpoint de
depósito con bloqueo a nivel de fila.

<details>
<summary><b>El detalle que más me costó: transferencias concurrentes sin corromper saldos</b></summary>

<br/>

Una transferencia no puede ser una sola petición HTTP: si dos usuarios mueven plata de
la misma cuenta al mismo tiempo, el saldo se rompe. La resolví partiendo el flujo en dos
— una respuesta inmediata `202 Accepted` y un procesamiento asíncrono con
`SELECT ... FOR UPDATE` ordenado por ID para evitar interbloqueos.

Este diagrama está tomado de la documentación que escribí en el repo,
[`cococash-wallet-ms/docs/transfer-flow.md`](https://github.com/TeamC-swarchqua/cococash/blob/main/cococash-wallet-ms/docs/transfer-flow.md):

```mermaid
sequenceDiagram
    participant User
    participant WalletMS as Wallet MS
    participant RDS as PostgreSQL
    participant SNS as SNS
    participant SQS as SQS
    participant Consumer as Consumer

    User->>WalletMS: POST /transfers
    WalletMS->>RDS: Validar cuentas y saldo
    WalletMS->>RDS: INSERT transfer (PENDING)
    WalletMS->>SNS: transfer.initiated
    WalletMS-->>User: 202 Accepted

    SNS->>SQS: Entrega asíncrona
    Consumer->>SQS: Long polling (20s)

    Note over Consumer,RDS: Bloqueo a nivel de fila
    Consumer->>RDS: BEGIN
    Consumer->>RDS: SELECT ... FOR UPDATE (ordenado por ID)

    alt Saldo suficiente
        Consumer->>RDS: Debitar origen, acreditar destino
        Consumer->>RDS: COMMIT (COMPLETED)
        Consumer->>SNS: transfer.completed
    else Saldo insuficiente
        Consumer->>RDS: COMMIT (FAILED)
        Consumer->>SNS: transfer.failed
    end
```

Si el consumidor falla, el mensaje vuelve a SQS por *visibility timeout*; a los tres
intentos cae en una DLQ en vez de perderse.

</details>

`TypeScript` `Go` `Terraform` `PostgreSQL` `DynamoDB` `AWS: ECS Fargate · Lambda · RDS · SNS · SQS · Cognito · S3 · EventBridge · ALB · VPC` `Docker`

---

## 🍳 [Recipy](https://github.com/recipy-swarch/recipy) — red social de recetas en microservicios

Dieciocho servicios, cuatro bases de datos, proxy inverso, y despliegue en Docker Compose
y Kubernetes.

```mermaid
flowchart TB
    U([Usuario]) --> RP["recipy-rp<br/>proxy inverso"]
    RP --> F["recipy-frontend<br/>Next.js"]
    F --> AG["recipy-ag<br/>API Gateway · NestJS"]
    AG -->|GraphQL| RMS["recipe-ms<br/>FastAPI · GraphQL"]
    AG --> UMS[userauth-ms]
    AG --> IMS[image-ms]
    AG --> MB["mail-broker<br/>RabbitMQ"]
    RMS --> CACHE["recipy-cache<br/>FastAPI · Redis"]
    RMS --> MDB[(MongoDB)]
    UMS --> PG[(PostgreSQL)]
    MB --> MMS[mail-ms]

    classDef mio fill:#238636,stroke:#2ea043,stroke-width:2px,color:#fff
    classDef parcial fill:#9e6a03,stroke:#bb8009,stroke-width:2px,color:#fff
    classDef equipo fill:#484f58,stroke:#6e7681,color:#fff
    class RMS,CACHE mio
    class AG,F parcial
    class RP,UMS,IMS,MB,MMS equipo
```

<sub>🟩 escrito por mí · 🟨 aportes puntuales · ⬜ del resto del equipo</sub>

Fui responsable del **microservicio de recetas** (`recipe-ms`): API en FastAPI con GraphQL sobre MongoDB, validación de JWT, y los endpoints
de recetas, comentarios y likes. Escribí también el **servicio de caché** sobre Redis
(`recipy-cache`, 82 de 82 líneas) y lo integré con `recipe-ms`.

Aportes puntuales en la capa de servicios del frontend en Next.js, el enrutamiento del
gateway en NestJS, el `docker-compose` y los scripts de inicialización de MongoDB.

`Python` `FastAPI` `GraphQL` `MongoDB` `Redis` `TypeScript` `Next.js` `NestJS` `Docker` `Kubernetes`

---

## 🔧 Proyectos más chicos

**[devops-reto-automatizacion-entorno](https://github.com/GorenNM/devops-reto-automatizacion-entorno)** — repo propio. Una app Node.js contenerizada, un módulo de Terraform que construye y publica la imagen, y un script de gestión del entorno Docker. Infraestructura como código escrita de punta a punta por mí. · `Terraform` `Docker` `Node.js` `Bash`

**[gopdfsuit-qua](https://github.com/TeamC-swarchqua/gopdfsuit-qua)** — ⚠️ **es un fork** de [GoPdfSuit](https://github.com/chinmay-sawant/gopdfsuit) (MIT); Mi aporte encima: la suite de pruebas de integración y end-to-end (Playwright en el frontend, pruebas en Go en el backend), el reemplazo de Google OAuth por un servicio de autenticación propio en el middleware, y el análisis de seguridad y mantenibilidad — ZAP, Trivy, gosec, golangci-lint y modelo de amenazas. · `Go` `Playwright` `Vitest` `SAST/DAST`

---

## 🧰 Con lo que trabajo

|  |  |
|---|---|
| **Lenguajes** | Python · Go · TypeScript / JavaScript · SQL · Bash |
| **Nube** | AWS — ECS Fargate · Lambda · RDS · DynamoDB · S3 · SNS · SQS · EventBridge · Cognito · API Gateway · ALB · VPC · CloudWatch · ECR |
| **Datos** | PostgreSQL · MongoDB · DynamoDB · Redis |
| **Infraestructura** | Terraform · Docker · Docker Compose · Kubernetes · Nginx |
| **Backend** | FastAPI · GraphQL · Express · NestJS · arquitecturas orientadas a eventos |
| **Frontend** | Next.js · React |

---

## 📬 Hablemos

**femurcia6@gmail.com** · **[linkedin.com/in/fabio-murcia](https://linkedin.com/in/fabio-murcia)**
