# Plantillas de Configuración - Spring Boot

Este documento contiene las plantillas de archivos de configuración para el proyecto.

## Tabla de Contenidos

1. [application.properties](#applicationproperties)
2. [application-dev.properties](#application-devproperties)
3. [application-prod.properties](#application-prodproperties)
4. [application-test.properties](#application-testproperties)
5. [logback-spring.xml](#logback-springxml)
6. [.gitignore](#gitignore)

---

## application.properties

```properties
# ===================================================================
# CONFIGURACIÓN COMÚN
# ===================================================================

spring.application.name={{PROJECT_NAME}}

# Puerto del servidor
server.port=8080

# Perfil activo por defecto
spring.profiles.active=dev

# ===================================================================
# JPA / HIBERNATE
# ===================================================================

# Estrategia de DDL
spring.jpa.hibernate.ddl-auto=update

# Propiedades de Hibernate
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true

# ===================================================================
# JACKSON
# ===================================================================

# Formato de fechas
spring.jackson.date-format=yyyy-MM-dd
spring.jackson.time-zone=America/Lima
spring.jackson.serialization.write-dates-as-timestamps=false

# ===================================================================
# OPENAPI / SWAGGER
# ===================================================================

springdoc.api-docs.path=/v3/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operationsSorter=method
springdoc.swagger-ui.tagsSorter=alpha

# ===================================================================
# ACTUATOR
# ===================================================================

management.endpoints.web.exposure.include=health,info
management.endpoint.health.show-details=when_authorized
```

---

## application-dev.properties

```properties
# ===================================================================
# PERFIL: DESARROLLO
# ===================================================================

# ===================================================================
# BASE DE DATOS - PostgreSQL (Neon DB)
# ===================================================================

spring.datasource.url=${NEON_DB_URL}
spring.datasource.username=${NEON_DB_USER}
spring.datasource.password=${NEON_DB_PASSWORD}
spring.datasource.driver-class-name=org.postgresql.Driver

# Pool de conexiones
spring.datasource.hikari.maximum-pool-size=5
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.idle-timeout=30000

# ===================================================================
# JPA / HIBERNATE
# ===================================================================

spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=update

# ===================================================================
# JWT
# ===================================================================

jwt.secret=${JWT_SECRET}
jwt.expiration=${JWT_EXPIRATION:86400000}

# ===================================================================
# LOGGING
# ===================================================================

logging.level.root=INFO
logging.level.{{PACKAGE}}=DEBUG
logging.level.org.springframework.security=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

# ===================================================================
# SWAGGER
# ===================================================================

springdoc.swagger-ui.enabled=true
```

---

## application-prod.properties

```properties
# ===================================================================
# PERFIL: PRODUCCIÓN
# ===================================================================

# ===================================================================
# BASE DE DATOS - PostgreSQL (Neon DB)
# ===================================================================

spring.datasource.url=${NEON_DB_URL}
spring.datasource.username=${NEON_DB_USER}
spring.datasource.password=${NEON_DB_PASSWORD}
spring.datasource.driver-class-name=org.postgresql.Driver

# Pool de conexiones - Producción
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=60000

# ===================================================================
# JPA / HIBERNATE
# ===================================================================

spring.jpa.show-sql=false
spring.jpa.hibernate.ddl-auto=validate

# ===================================================================
# JWT
# ===================================================================

jwt.secret=${JWT_SECRET}
jwt.expiration=${JWT_EXPIRATION:86400000}

# ===================================================================
# LOGGING
# ===================================================================

logging.level.root=WARN
logging.level.{{PACKAGE}}=INFO
logging.level.org.springframework.security=WARN

# ===================================================================
# SWAGGER - Protegido en producción
# ===================================================================

springdoc.swagger-ui.enabled=false
```

---

## application-test.properties

```properties
# ===================================================================
# PERFIL: TESTING
# ===================================================================

# ===================================================================
# BASE DE DATOS - H2 en memoria
# ===================================================================

spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# Consola H2 (opcional para debugging)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# ===================================================================
# JPA / HIBERNATE
# ===================================================================

spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

# ===================================================================
# JWT
# ===================================================================

jwt.secret=test-secret-key-for-testing-purposes-must-be-at-least-64-characters-long-for-hs512
jwt.expiration=86400000

# ===================================================================
# LOGGING
# ===================================================================

logging.level.root=INFO
logging.level.{{PACKAGE}}=DEBUG
logging.level.org.springframework.security=DEBUG
```

---

## logback-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- Propiedades -->
    <property name="LOG_PATH" value="${LOG_PATH:-logs}"/>
    <property name="LOG_FILE" value="${LOG_FILE:-application}"/>
    
    <!-- Patrón de log -->
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>

    <!-- Appender para consola -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- Appender para archivo - Desarrollo -->
    <springProfile name="dev">
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${LOG_PATH}/${LOG_FILE}.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${LOG_PATH}/${LOG_FILE}-%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>7</maxHistory>
                <totalSizeCap>100MB</totalSizeCap>
            </rollingPolicy>
            <encoder>
                <pattern>${LOG_PATTERN}</pattern>
                <charset>UTF-8</charset>
            </encoder>
        </appender>

        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
            <appender-ref ref="FILE"/>
        </root>

        <!-- Niveles específicos para desarrollo -->
        <logger name="{{PACKAGE}}" level="DEBUG"/>
        <logger name="org.springframework.security" level="DEBUG"/>
        <logger name="org.hibernate.SQL" level="DEBUG"/>
        <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>
    </springProfile>

    <!-- Appender para archivo - Producción -->
    <springProfile name="prod">
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${LOG_PATH}/${LOG_FILE}.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${LOG_PATH}/${LOG_FILE}-%d{yyyy-MM-dd}.log.gz</fileNamePattern>
                <maxHistory>30</maxHistory>
                <totalSizeCap>1GB</totalSizeCap>
            </rollingPolicy>
            <encoder>
                <pattern>${LOG_PATTERN}</pattern>
                <charset>UTF-8</charset>
            </encoder>
        </appender>

        <root level="WARN">
            <appender-ref ref="CONSOLE"/>
            <appender-ref ref="FILE"/>
        </root>

        <!-- Niveles específicos para producción -->
        <logger name="{{PACKAGE}}" level="INFO"/>
        <logger name="org.springframework.security" level="WARN"/>
    </springProfile>

    <!-- Configuración para tests -->
    <springProfile name="test">
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
        
        <logger name="{{PACKAGE}}" level="DEBUG"/>
    </springProfile>

</configuration>
```

---

## .gitignore

```gitignore
# ===================================================================
# COMPILADOS Y BUILD
# ===================================================================
target/
!.mvn/wrapper/maven-wrapper.jar
!**/src/main/**/target/
!**/src/test/**/target/

# ===================================================================
# IDE - IntelliJ IDEA
# ===================================================================
.idea/
*.iws
*.iml
*.ipr

# ===================================================================
# IDE - Eclipse
# ===================================================================
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

# ===================================================================
# IDE - NetBeans
# ===================================================================
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/
!**/src/main/**/build/
!**/src/test/**/build/

# ===================================================================
# IDE - VS Code
# ===================================================================
.vscode/

# ===================================================================
# LOGS
# ===================================================================
logs/
*.log

# ===================================================================
# SISTEMA OPERATIVO
# ===================================================================
.DS_Store
Thumbs.db

# ===================================================================
# VARIABLES DE ENTORNO Y SECRETS
# ===================================================================
.env
.env.local
.env.*.local
*.env

# ===================================================================
# DOCKER
# ===================================================================
docker-compose.override.yml

# ===================================================================
# OTROS
# ===================================================================
*.swp
*~
.cache
```
