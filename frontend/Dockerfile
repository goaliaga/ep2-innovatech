# ==========================================
# ETAPA 1: Construcción
# Usamos Node para compilar el proyecto React
# ==========================================
FROM node:20-alpine AS build

# Directorio de trabajo dentro del contenedor
WORKDIR /app

# Copiamos primero solo los archivos de dependencias
# Esto aprovecha el cache de Docker: si no cambian
# las dependencias, no las reinstala en cada build
COPY package.json package-lock.json ./

# Instalamos las dependencias
RUN npm install

# Copiamos el resto del código
COPY . .

# Compilamos el proyecto (genera carpeta /app/dist)
RUN npm run build

# ==========================================
# ETAPA 2: Producción
# Usamos Nginx para servir los archivos estáticos
# Imagen mucho más liviana que Node
# ==========================================
FROM nginx:1.19.0-alpine

# Copiamos los archivos compilados desde la etapa anterior
# Solo copiamos el resultado final, no el código fuente
COPY --from=build /app/dist/ /usr/share/nginx/html

# Exponemos el puerto 80
EXPOSE 80