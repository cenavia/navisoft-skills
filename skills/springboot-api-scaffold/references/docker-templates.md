# Plantillas Docker - Spring Boot

Este documento contiene las plantillas para containerización del proyecto.

## Tabla de Contenidos

1. [Dockerfile](#dockerfile)
2. [docker-compose.yml](#docker-composeyml)
3. [.dockerignore](#dockerignore)

---

## Dockerfile

### Multi-stage Build (Recomendado)

```dockerfile
# ===================================================================
# STAGE 1: Build
# ===================================================================
FROM eclipse-temurin:21-jdk-alpine AS build

WORKDIR /app

# Copiar archivos de Maven
COPY pom.xml .
COPY .mvn .mvn
COPY mvnw .

# Dar permisos de ejecución al wrapper
RUN chmod +x mvnw

# Descargar dependencias (aprovecha cache de Docker)
RUN ./mvnw dependency:go-offline -B

# Copiar código fuente
COPY src src

# Construir la aplicación
RUN ./mvnw package -DskipTests

# ===================================================================
# STAGE 2: Runtime
# ===================================================================
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Crear usuario no-root para seguridad
RUN addgroup -S spring && adduser -S spring -G spring

# Copiar JAR desde stage de build
COPY --from=build /app/target/*.jar app.jar

# Cambiar al usuario no-root
USER spring:spring

# Puerto de la aplicación
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# Variables de entorno con valores por defecto
ENV JAVA_OPTS="-Xmx512m -Xms256m"

# Comando de inicio
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### Dockerfile Simplificado (Desarrollo)

```dockerfile
FROM eclipse-temurin:21-jdk-alpine

WORKDIR /app

# Copiar JAR pre-construido
COPY target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## docker-compose.yml

### Configuración Completa

```yaml
version: '3.8'

services:
  # ===================================================================
  # APLICACIÓN SPRING BOOT
  # ===================================================================
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: {{PROJECT_NAME}}-app
    ports:
      - "8080:8080"
    environment:
      # Perfil de Spring
      - SPRING_PROFILES_ACTIVE=dev
      
      # Base de datos
      - NEON_DB_URL=${NEON_DB_URL}
      - NEON_DB_USER=${NEON_DB_USER}
      - NEON_DB_PASSWORD=${NEON_DB_PASSWORD}
      
      # JWT
      - JWT_SECRET=${JWT_SECRET}
      - JWT_EXPIRATION=${JWT_EXPIRATION:-86400000}
      
      # JVM Options
      - JAVA_OPTS=-Xmx512m -Xms256m
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - {{PROJECT_NAME}}-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

  # ===================================================================
  # BASE DE DATOS POSTGRESQL (Local)
  # ===================================================================
  postgres:
    image: postgres:16-alpine
    container_name: {{PROJECT_NAME}}-postgres
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB={{PROJECT_NAME}}
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=admin123
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    networks:
      - {{PROJECT_NAME}}-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d {{PROJECT_NAME}}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ===================================================================
  # PGADMIN (Opcional - Interfaz de administración)
  # ===================================================================
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: {{PROJECT_NAME}}-pgadmin
    ports:
      - "5050:80"
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=admin123
    depends_on:
      - postgres
    networks:
      - {{PROJECT_NAME}}-network
    restart: unless-stopped
    profiles:
      - tools

networks:
  {{PROJECT_NAME}}-network:
    driver: bridge

volumes:
  postgres_data:
    driver: local
```

### docker-compose.override.yml (Desarrollo Local)

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      # Hot reload de código (desarrollo)
      - ./src:/app/src
      - ./pom.xml:/app/pom.xml
    environment:
      - SPRING_PROFILES_ACTIVE=dev
      - SPRING_DEVTOOLS_RESTART_ENABLED=true
    ports:
      - "8080:8080"
      - "5005:5005"  # Puerto de debug
    command: >
      sh -c "
        ./mvnw spring-boot:run 
        -Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005'
      "
```

### docker-compose.prod.yml (Producción)

```yaml
version: '3.8'

services:
  app:
    image: {{REGISTRY}}/{{PROJECT_NAME}}:${VERSION:-latest}
    container_name: {{PROJECT_NAME}}-app
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - NEON_DB_URL=${NEON_DB_URL}
      - NEON_DB_USER=${NEON_DB_USER}
      - NEON_DB_PASSWORD=${NEON_DB_PASSWORD}
      - JWT_SECRET=${JWT_SECRET}
      - JWT_EXPIRATION=${JWT_EXPIRATION}
      - JAVA_OPTS=-Xmx1024m -Xms512m -XX:+UseG1GC
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1536M
        reservations:
          cpus: '1'
          memory: 512M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    networks:
      - {{PROJECT_NAME}}-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 90s

networks:
  {{PROJECT_NAME}}-network:
    external: true
```

---

## .dockerignore

```dockerignore
# ===================================================================
# ARCHIVOS DE BUILD Y DEPENDENCIAS
# ===================================================================
target/
!target/*.jar
.mvn/
mvnw
mvnw.cmd

# ===================================================================
# IDE Y EDITORES
# ===================================================================
.idea/
*.iml
*.iws
*.ipr
.vscode/
.project
.classpath
.settings/

# ===================================================================
# GIT
# ===================================================================
.git/
.gitignore
.gitattributes

# ===================================================================
# DOCKER
# ===================================================================
Dockerfile*
docker-compose*.yml
.docker/

# ===================================================================
# DOCUMENTACIÓN Y OTROS
# ===================================================================
README.md
CHANGELOG.md
LICENSE
docs/

# ===================================================================
# LOGS Y TEMPORALES
# ===================================================================
logs/
*.log
*.tmp
*.swp
*~

# ===================================================================
# VARIABLES DE ENTORNO
# ===================================================================
.env
.env.*
*.env

# ===================================================================
# TESTS
# ===================================================================
src/test/
```

---

## Comandos Útiles

### Build y Ejecución

```bash
# Build de imagen
docker build -t {{PROJECT_NAME}}:latest .

# Ejecutar con docker-compose
docker-compose up -d

# Ejecutar en modo desarrollo
docker-compose -f docker-compose.yml -f docker-compose.override.yml up

# Ejecutar en producción
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Ver logs
docker-compose logs -f app

# Detener servicios
docker-compose down

# Detener y eliminar volúmenes
docker-compose down -v
```

### Debugging

```bash
# Acceder al contenedor
docker exec -it {{PROJECT_NAME}}-app /bin/sh

# Ver estadísticas
docker stats {{PROJECT_NAME}}-app

# Verificar health check
docker inspect --format='{{json .State.Health}}' {{PROJECT_NAME}}-app
```

### Limpieza

```bash
# Eliminar contenedores detenidos
docker container prune

# Eliminar imágenes no utilizadas
docker image prune

# Limpieza completa
docker system prune -a --volumes
```
