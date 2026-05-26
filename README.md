# 🚚 Innovatech Chile — Sistema de Gestión de Despachos y Ventas

Plataforma de microservicios contenedorizada y desplegada en AWS para la gestión de despachos y ventas de Innovatech Chile. Desarrollada como parte de la Evaluación Parcial N°2 del curso ISY1101 - Introducción a Herramientas DevOps.

---

## 🏗️ Arquitectura

```
Internet
    ↓
[EC2 Frontend - React/Nginx] puerto 80 (IP pública)
    ↓ IP privada
[EC2 Backend Despachos - Spring Boot] puerto 8081
[EC2 Backend Ventas - Spring Boot]    puerto 8080
    ↓ IP privada
[RDS MySQL 8.0] puerto 3306
```

Cada servicio corre en su propia instancia EC2 dentro de una VPC privada en AWS. Solo el frontend es accesible desde internet. Los backends y la base de datos solo aceptan tráfico desde la capa anterior gracias a los Security Groups configurados por capas.

---

## 📁 Estructura del repositorio

```
ep2-innovatech/
├── frontend/                    # Aplicación React + Vite
│   ├── src/                     # Código fuente React
│   ├── Dockerfile               # Multi-stage: Node (build) + Nginx (serve)
│   ├── nginx.conf               # Configuración proxy inverso a backends
│   └── docker-compose.yml       # Composición del servicio frontend
├── backend-despachos/           # API REST Spring Boot - Gestión de despachos
│   ├── src/                     # Código fuente Java
│   ├── Dockerfile               # Multi-stage: Maven (build) + JRE (run)
│   └── docker-compose.yml       # Composición del servicio backend despachos
├── backend-ventas/              # API REST Spring Boot - Gestión de ventas
│   ├── src/                     # Código fuente Java
│   ├── Dockerfile               # Multi-stage: Maven (build) + JRE (run)
│   └── docker-compose.yml       # Composición del servicio backend ventas
└── .github/
    └── workflows/
        ├── cicd-frontend.yml           # Pipeline CI/CD frontend
        ├── cicd-backend-despachos.yml  # Pipeline CI/CD backend despachos
        └── cicd-backend-ventas.yml     # Pipeline CI/CD backend ventas
```

---

## 🐳 Dockerfiles — Multi-stage Build

Cada servicio usa **multi-stage build** para optimizar el tamaño de las imágenes finales y mejorar la seguridad.

### Frontend (React + Nginx)
- **Etapa 1 (build):** Imagen `node:20-alpine` compila el proyecto React con `npm run build`
- **Etapa 2 (serve):** Imagen `nginx:1.19.0-alpine` sirve los archivos estáticos compilados
- La imagen final no contiene Node.js ni el código fuente, solo los archivos HTML/CSS/JS

### Backend Despachos y Ventas (Spring Boot)
- **Etapa 1 (build):** Imagen `maven:3.8.8-eclipse-temurin-17` compila el proyecto y genera el JAR
- **Etapa 2 (run):** Imagen `eclipse-temurin:17-jre` ejecuta el JAR con un usuario no root
- La imagen final no contiene Maven ni el código fuente, solo el JAR ejecutable
- Se crea un usuario `appuser` sin privilegios para ejecutar la aplicación (seguridad)

---

## 🔧 Variables de entorno

Los backends requieren un archivo `.env` en `/home/ec2-user/app/` de cada EC2:

```env
DB_ENDPOINT=<endpoint-rds>.us-east-1.rds.amazonaws.com
DB_PORT=3306
DB_NAME=<nombre_base_de_datos>
DB_USERNAME=admin
DB_PASSWORD=<password>
```

---

## 🚀 Cómo levantar localmente

### Frontend
```bash
cd frontend
npm install
npm run dev
```
Disponible en `http://localhost:5173`

### Backend (requiere MySQL corriendo)
```bash
# Levantar MySQL con Docker
docker run -d --name mysql-local \
  -e MYSQL_ROOT_PASSWORD=admin123 \
  -e MYSQL_DATABASE=despachos_db \
  -p 3306:3306 mysql:8

# Correr backend despachos
cd backend-despachos
./mvnw spring-boot:run
```

### Con Docker Compose (cada servicio por separado)
```bash
# Frontend
cd frontend
docker-compose up -d

# Backend Despachos
cd backend-despachos
docker-compose up -d

# Backend Ventas
cd backend-ventas
docker-compose up -d
```

---

## ⚙️ Pipeline CI/CD

Cada servicio tiene su propio workflow en GitHub Actions que se activa automáticamente con un `push` a la rama **`deploy`** cuando hay cambios en su carpeta correspondiente.

### Flujo del pipeline:
```
git push origin deploy
        ↓
GitHub Actions detecta cambios en la carpeta del servicio
        ↓
1. Checkout del repositorio
2. Login en Docker Hub
3. Build de la imagen Docker (multi-stage)
4. Tag con versión y :latest
5. Push a Docker Hub
        ↓
Deploy manual en EC2:
cd /home/ec2-user/app
docker-compose pull
docker-compose up -d
```

### Secrets requeridos en GitHub:
| Secret | Descripción |
|---|---|
| `DOCKERHUB_USERNAME` | Usuario de Docker Hub |
| `DOCKERHUB_TOKEN` | Token de acceso Docker Hub |
| `AWS_ACCESS_KEY_ID` | Credencial AWS |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS |
| `AWS_SESSION_TOKEN` | Token de sesión AWS Academy |

---

## 🔒 Seguridad — Security Groups

| Capa | Puerto | Origen |
|---|---|---|
| Frontend | 80 | 0.0.0.0/0 (internet) |
| Backend Despachos | 8081 | sg-frontend-ep2 |
| Backend Ventas | 8080 | sg-frontend-ep2 |
| RDS MySQL | 3306 | sg-backend-ep2 |

Solo el frontend es accesible desde internet. Los backends solo aceptan tráfico del frontend y la base de datos solo acepta tráfico de los backends.

---

## 🗄️ Persistencia de datos

Se utilizan **named volumes** en Docker para persistir los logs de cada backend:
- `despachos_logs` → logs del backend de despachos
- `ventas_logs` → logs del backend de ventas

Se eligió **named volume** en vez de bind mount porque:
- Docker administra el ciclo de vida del volumen automáticamente
- Los datos persisten aunque el contenedor sea eliminado y recreado
- No depende de rutas específicas del sistema de archivos del host

---

## 🛠️ Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| React + Vite | Frontend SPA |
| Spring Boot 3 + Java 17 | Backend REST API |
| MySQL 8 | Base de datos relacional |
| Docker + Docker Compose | Contenedorización |
| Nginx | Servidor web y proxy inverso |
| GitHub Actions | CI/CD automatizado |
| Amazon EC2 | Servidores en la nube |
| Amazon RDS | Base de datos administrada |
| Docker Hub | Registro de imágenes |
| AWS VPC | Red privada virtual |