# ==========================================
# ETAPA 1: Compilación
# Usamos Maven con Java 17 para compilar el proyecto
# ==========================================
FROM maven:3.8.8-eclipse-temurin-17 AS build

WORKDIR /app

# Copiamos primero solo el pom.xml
# Si no cambian las dependencias, Docker usa el cache
COPY pom.xml .
RUN mvn dependency:go-offline

# Copiamos el código fuente
COPY src ./src

# Compilamos y generamos el JAR (saltamos los tests)
RUN mvn clean package -DskipTests

# ==========================================
# ETAPA 2: Ejecución
# Solo necesitamos el JRE, no el JDK completo
# Imagen mucho más liviana
# ==========================================
FROM eclipse-temurin:17-jre

WORKDIR /app

# Copiamos solo el JAR desde la etapa anterior
COPY --from=build /app/target/*.jar app.jar

# Usuario no root por seguridad
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser
USER appuser

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar"]