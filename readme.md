# ⚙️ Innovatech — Backend Microservicios (Ventas + Despachos)

> Dos microservicios Spring Boot para gestión de ventas y despachos, orquestados en AWS ECS con pipeline CI/CD automatizado.

![Deploy](https://img.shields.io/badge/Deploy-AWS%20ECS%20Fargate-orange?logo=amazon-aws)
![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-blue?logo=github-actions)
![Tech](https://img.shields.io/badge/Tech-Spring%20Boot%20%2B%20Java%2021-6DB33F?logo=spring)
![DB](https://img.shields.io/badge/DB-MySQL%208.0-4479A1?logo=mysql)

---

## 📋 Descripción

Este repositorio contiene dos microservicios backend independientes que exponen APIs REST para la aplicación Innovatech Chile:

- **back-Ventas_SpringBoot** — Gestión de órdenes de compra/ventas (puerto `8080`)
- **back-Despachos_SpringBoot** — Gestión de órdenes de despacho y entregas (puerto `8081`)

Ambos microservicios comparten una base de datos MySQL y son desplegados de forma independiente en AWS ECS Fargate.

---

## 🏗️ Arquitectura de Despliegue

```
┌─────────────────────────────────────────────────────┐
│              ALB Interno (innovatech-internal-alb)  │
│         HTTP:8080 ──────────► ventas-tg             │
│         HTTP:8081 ──────────► despachos-tg          │
└──────────────┬─────────────────────┬────────────────┘
               │                     │
               ▼                     ▼
┌──────────────────────┐  ┌──────────────────────────┐
│  innovatech-ventas   │  │  innovatech-despachos     │
│  ECS Task · :8080    │  │  ECS Task · :8081         │
│  Spring Boot + Java  │  │  Spring Boot + Java        │
│  subred privada      │  │  subred privada            │
└──────────┬───────────┘  └────────────┬──────────────┘
           │                           │
           └─────────────┬─────────────┘
                         ▼
               ┌──────────────────┐
               │   MySQL 8.0      │
               │   Puerto 3306    │
               │   innovatech DB  │
               └──────────────────┘
```

---

## 🛠️ Tecnologías

| Tecnología | Versión | Uso |
|---|---|---|
| Java | OpenJDK 21 | Lenguaje de programación |
| Spring Boot | 3+ | Framework backend REST |
| Maven | 3.9.9 | Gestión de dependencias y build |
| MySQL | 8.0 | Base de datos relacional |
| Docker | — | Contenedorización multi-stage |
| AWS ECS Fargate | — | Orquestación en la nube |
| Amazon ECR | — | Registro de imágenes Docker |
| GitHub Actions | — | Pipeline CI/CD |

---

## 📁 Estructura del Repositorio

```
backendDev/
├── back-Ventas_SpringBoot/
│   └── Springboot-API-REST/
│       ├── src/
│       │   └── main/java/...    # Código fuente Spring Boot Ventas
│       ├── pom.xml              # Dependencias Maven
│       └── Dockerfile           # Imagen Docker multi-stage
│
├── back-Despachos_SpringBoot/
│   └── Springboot-API-REST-DESPACHO/
│       ├── src/
│       │   └── main/java/...    # Código fuente Spring Boot Despachos
│       ├── pom.xml              # Dependencias Maven
│       └── Dockerfile           # Imagen Docker multi-stage
│
└── docker-compose.yml           # Levantamiento local completo
```

---

## 🔌 Endpoints disponibles

### Servicio Ventas (`:8080`)

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/api/v1/ventas` | Listar todas las órdenes de compra |
| `POST` | `/api/v1/ventas` | Crear nueva orden de compra |
| `GET` | `/api/v1/ventas/{id}` | Obtener orden por ID |
| `PUT` | `/api/v1/ventas/{id}` | Actualizar orden |
| `DELETE` | `/api/v1/ventas/{id}` | Eliminar orden |

### Servicio Despachos (`:8081`)

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/api/v1/despachos` | Listar todos los despachos |
| `POST` | `/api/v1/despachos` | Crear nueva orden de despacho |
| `GET` | `/api/v1/despachos/{id}` | Obtener despacho por ID |
| `PUT` | `/api/v1/despachos/{id}` | Actualizar despacho |

---

## ⚡ Ejecución Local

### Prerequisitos

- Docker Desktop instalado
- Git

### 1. Clonar el repositorio

```bash
git clone https://github.com/zhet0M/backendDev.git
cd backendDev
```

### 2. Levantar con Docker Compose

```bash
docker compose up --build
```

Esto levanta automáticamente:
- MySQL en puerto `3306`
- Backend Ventas en `http://localhost:8080`
- Backend Despachos en `http://localhost:8081`

### 3. Verificar que los servicios estén corriendo

```bash
curl http://localhost:8080/api/v1/ventas
# Respuesta esperada: [] (lista vacía)

curl http://localhost:8081/api/v1/despachos
# Respuesta esperada: [] (lista vacía)
```

### 4. Detener los servicios

```bash
docker compose down
```

---

## 🐳 Dockerfiles (Multi-stage Build)

Ambos microservicios utilizan el mismo patrón multi-stage para separar compilación y ejecución:

```dockerfile
# Stage 1: Compilación con Maven
FROM maven:3.9.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Imagen de ejecución liviana
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=build /app/target/*.jar app.jar
RUN chown -R appuser:appgroup /app
USER appuser
EXPOSE 8080   # (8081 en despachos)
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## ☁️ Despliegue en AWS ECS

### Rama de despliegue

El pipeline CI/CD se activa automáticamente al hacer push a la rama `deploy`:

```bash
git checkout deploy
git merge main
git push origin deploy
```

### Pipeline CI/CD (GitHub Actions)

```
push → deploy branch
        │
        ▼
  1. Descargar repositorio (actions/checkout@v4)
        │
        ▼
  2. Configurar credenciales AWS (aws-actions/configure-aws-credentials@v4)
        │
        ▼
  3. Login ECR (aws-actions/amazon-ecr-login@v2)
        │
        ▼
  4. Definir nombres de imágenes (VENTAS_IMAGE y DESPACHOS_IMAGE)
        │
        ▼
  5. Build y push Ventas Backend (docker/build-push-action@v6)
        │
        ▼
  6. Build y push Despachos Backend (docker/build-push-action@v6)
        │
        ▼
  7. Deploy a ECS Ventas (aws ecs update-service)
        │
        ▼
  8. Deploy a ECS Despachos (aws ecs update-service)
        │
        ▼
  ✅ Nuevas tareas ECS levantadas
```

### GitHub Secrets requeridos

Configura los siguientes secrets en **Settings → Secrets and variables → Actions**:

| Secret | Descripción |
|---|---|
| `AWS_ACCESS_KEY_ID` | Clave de acceso AWS Academy |
| `AWS_SECRET_ACCESS_KEY` | Clave secreta AWS |
| `AWS_SESSION_TOKEN` | Token de sesión temporal |
| `AWS_REGION` | Región (`us-east-1`) |
| `BACKEND_INSTANCE_ID` | ID de instancia ECS (si aplica) |
| `MYSQL_DATABASE` | Nombre de la base de datos (`innovatech`) |
| `MYSQL_ROOT_PASSWORD` | Contraseña root MySQL |

### Variables de entorno en ECS (Task Definition)

Las Task Definitions de ECS inyectan las siguientes variables en cada contenedor:

```env
SPRING_DATASOURCE_URL=jdbc:mysql://<host>:3306/innovatech
SPRING_DATASOURCE_USERNAME=root
SPRING_DATASOURCE_PASSWORD=<secret>
```

---

## 🔒 Seguridad

- Las credenciales de AWS nunca se hardcodean en el código; se gestionan mediante **GitHub Secrets**.
- Las contraseñas de base de datos se inyectan como variables de entorno en la Task Definition de ECS.
- Los contenedores backend corren como usuario no-root (`appuser`) dentro del contenedor.
- Los servicios backend están en **subred privada** de AWS, sin IP pública asignada.
- El acceso externo a los backends solo es posible a través del **ALB interno** desde el frontend.

---

## 📊 Logs y Monitoreo

Los logs de ambos microservicios son enviados automáticamente a **Amazon CloudWatch Logs**:

| Servicio | Grupo de logs CloudWatch |
|---|---|
| Ventas | `/ecs/innovatech-ventas-td` |
| Despachos | `/ecs/innovatech-despachos-td` |

Para consultar logs localmente:

```bash
# Ver logs de un contenedor corriendo
docker logs innovatech-ventas-backend -f
docker logs innovatech-despachos-backend -f
```

---

## 👥 Integrantes

- Cristóbal Medina
- Matías Medina

**Asignatura:** Introducción a Herramientas DevOps — ISY1101  
**Sección:** 004D | DuocUC 2025