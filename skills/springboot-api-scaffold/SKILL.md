---
name: springboot-api-scaffold
description: |
  Genera esqueletos de proyectos API REST con Spring Boot siguiendo arquitectura en capas tradicional.
  Usar cuando el usuario solicite:
  (1) Crear/generar una API REST Spring Boot desde cero
  (2) Scaffolding de entidades JPA, servicios, controladores y repositorios
  (3) Estructura inicial de proyecto con arquitectura en capas (controller/service/repository/domain)
  (4) Configurar seguridad JWT con roles ADMIN y USER
  (5) Generar archivos base con MapStruct, validaciones Jakarta y documentación OpenAPI
  Soporta Java 21+, Spring Boot 3.5.x, PostgreSQL, MapStruct 1.6.x, JWT, y patrones enterprise.
---

# Spring Boot API REST Scaffold Generator

Skill para generar esqueletos de proyectos API REST con Spring Boot y Java siguiendo arquitectura en capas tradicional con énfasis en optimización JPA, seguridad JWT y separación clara de responsabilidades.

## Tabla de Contenidos

1. [Prerrequisito Obligatorio](#prerrequisito-obligatorio)
2. [Flujo de Trabajo Principal](#flujo-de-trabajo-principal)
3. [Análisis del Documento de Arquitectura](#análisis-del-documento-de-arquitectura)
4. [Generación de Estructura](#generación-de-estructura)
5. [Stack Tecnológico](#stack-tecnológico)
6. [Patrones de Código](#patrones-de-código)
7. [Plantillas de Código](#plantillas-de-código)
8. [Generación del README.md](#generación-del-readmemd)
9. [Validación Final](#validación-final)

---

## Prerrequisito Obligatorio

> **⚠️ IMPORTANTE**: Antes de iniciar cualquier generación de scaffold, **SIEMPRE preguntar al usuario**:
>
> 1. **Nombre del proyecto** (obligatorio)
> 2. **Group ID de Maven** (obligatorio)
> 3. **Entidades principales** del dominio (opcional, puede inferirse del documento)
>
> El nombre del proyecto se usará para:
> - `artifactId` en `pom.xml`
> - Carpeta raíz del proyecto
> - Referencias en `README.md`
> - Clase principal `*Application.java`
>
> **Preguntas sugeridas**:
> - "¿Cuál es el nombre del proyecto? (usar kebab-case, ej: `gestion-eventos-api`)"
> - "¿Cuál es el Group ID de Maven? (ej: `com.empresa.proyecto`)"

---

## Flujo de Trabajo Principal

```
0. PREGUNTAR nombre del proyecto y Group ID al usuario (OBLIGATORIO)
1. ANALIZAR documento de arquitectura → extraer entidades y requisitos
2. PLANIFICAR estructura de carpetas por capas técnicas
3. GENERAR configuración base (pom.xml, application.properties)
4. CREAR capas: controller, service, repository, domain, dto, mapper, exception
5. GENERAR módulo de seguridad JWT completo
6. IMPLEMENTAR entidades JPA con relaciones y métodos helper
7. CREAR servicios con interfaces e implementaciones
8. GENERAR controladores con documentación OpenAPI
9. CONFIGURAR MapStruct mappers
10. CREAR GlobalExceptionHandler y excepciones personalizadas
11. GENERAR estructura de tests con JUnit 5 y Mockito
12. GENERAR README.md con guía completa de desarrollo
13. VALIDAR coherencia del scaffold (compila, tests pasan)
```

---

## Análisis del Documento de Arquitectura

### Extraer del Documento

| Categoría | Qué buscar | Impacto en scaffold |
|-----------|------------|---------------------|
| **Tech Stack** | Versión Java, Spring Boot, DB | `pom.xml`, propiedades |
| **Entidades** | Agregados, entidades JPA | `domain/` estructura |
| **Relaciones** | ManyToOne, ManyToMany, etc. | Configuración JPA |
| **DTOs** | Request/Response patterns | `dto/` estructura |
| **Endpoints** | Operaciones CRUD, custom | `controller/` métodos |
| **Roles** | ADMIN, USER, permisos | `security/` configuración |
| **Validaciones** | Reglas de negocio | DTOs con Jakarta Validation |

### Checklist de Requisitos

```markdown
[ ] Versión Java especificada (21+)
[ ] Versión Spring Boot especificada (3.5.x)
[ ] Base de datos definida (PostgreSQL + H2 para tests)
[ ] Entidades JPA identificadas
[ ] Relaciones entre entidades definidas
[ ] Roles de seguridad definidos (ADMIN, USER)
[ ] Endpoints REST listados
[ ] Validaciones de entrada requeridas
[ ] Mensajes de error en español
[ ] Logging en español
```

---

## Generación de Estructura

### Estructura Base Arquitectura en Capas

```
{project-name}/
├── src/
│   ├── main/
│   │   ├── java/{group-id-path}/
│   │   │   ├── {ProjectName}Application.java     # Punto de entrada @SpringBootApplication
│   │   │   │
│   │   │   ├── controller/                       # Capa de Presentación
│   │   │   │   └── {Entity}Controller.java
│   │   │   │
│   │   │   ├── service/                          # Capa de Negocio
│   │   │   │   ├── I{Entity}Service.java         # Interface
│   │   │   │   └── {Entity}Service.java          # Implementación
│   │   │   │
│   │   │   ├── repository/                       # Capa de Acceso a Datos
│   │   │   │   └── {Entity}Repository.java
│   │   │   │
│   │   │   ├── domain/                           # Entidades JPA
│   │   │   │   ├── {Entity}.java
│   │   │   │   ├── Role.java
│   │   │   │   └── User.java
│   │   │   │
│   │   │   ├── dto/                              # Data Transfer Objects
│   │   │   │   ├── {Entity}RequestDto.java
│   │   │   │   ├── {Entity}ResponseDto.java
│   │   │   │   └── {Entity}SummaryDto.java
│   │   │   │
│   │   │   ├── mapper/                           # MapStruct Mappers
│   │   │   │   └── {Entity}Mapper.java
│   │   │   │
│   │   │   ├── exception/                        # Manejo de Excepciones
│   │   │   │   ├── GlobalExceptionHandler.java
│   │   │   │   └── ResourceNotFoundException.java
│   │   │   │
│   │   │   ├── data/                             # Cargadores de Datos Iniciales
│   │   │   │   └── DataLoader.java
│   │   │   │
│   │   │   └── security/                         # Módulo de Seguridad JWT
│   │   │       ├── config/
│   │   │       │   ├── SecurityConfig.java
│   │   │       │   └── OpenApiConfig.java
│   │   │       ├── controller/
│   │   │       │   └── AuthController.java
│   │   │       ├── dto/
│   │   │       │   ├── LoginDto.java
│   │   │       │   ├── RegisterDto.java
│   │   │       │   └── JwtAuthResponseDto.java
│   │   │       ├── jwt/
│   │   │       │   ├── JwtGenerator.java
│   │   │       │   ├── JwtAuthenticationFilter.java
│   │   │       │   └── JwtAuthEntryPoint.java
│   │   │       └── service/
│   │   │           └── UserDetailsServiceImpl.java
│   │   │
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── application-dev.properties
│   │       ├── application-prod.properties
│   │       ├── application-test.properties
│   │       └── logback-spring.xml
│   │
│   └── test/
│       └── java/{group-id-path}/
│           ├── controller/
│           │   └── {Entity}ControllerTest.java
│           └── service/
│               └── {Entity}ServiceTest.java
│
├── pom.xml
├── Dockerfile
├── docker-compose.yml
├── .gitignore
└── README.md
```

### Reglas de Nomenclatura

| Tipo | Patrón | Ejemplo |
|------|--------|---------|
| **Entidad** | `{Name}.java` | `Event.java` |
| **Controlador** | `{Name}Controller.java` | `EventController.java` |
| **Interfaz Servicio** | `I{Name}Service.java` | `IEventService.java` |
| **Implementación Servicio** | `{Name}Service.java` | `EventService.java` |
| **Repositorio** | `{Name}Repository.java` | `EventRepository.java` |
| **DTO Entrada** | `{Name}RequestDto.java` | `EventRequestDto.java` |
| **DTO Salida** | `{Name}ResponseDto.java` | `EventResponseDto.java` |
| **DTO Resumen** | `{Name}SummaryDto.java` | `EventSummaryDto.java` |
| **Mapper** | `{Name}Mapper.java` | `EventMapper.java` |
| **Excepción** | `{Name}Exception.java` | `ResourceNotFoundException.java` |
| **Test** | `{Name}Test.java` | `EventServiceTest.java` |
| **Tabla BD** | snake_case plural | `events`, `event_speakers` |
| **Paquetes** | minúsculas sin guiones | `com.gestion.eventos.api` |

### Convenciones de Idioma

| Elemento | Idioma | Ejemplo |
|----------|--------|---------|
| Código (clases, métodos, variables) | **Inglés** | `findById`, `EventService` |
| Documentación y comentarios | **Español** | `// Buscar evento por ID` |
| Mensajes de log | **Español** | `logger.info("Evento guardado")` |
| Mensajes de validación | **Español** | `@NotBlank(message = "El nombre es obligatorio")` |
| Mensajes de error | **Español** | `"Evento no encontrado con id: " + id` |
| Documentación OpenAPI | **Español** | `@Tag(name = "Eventos")` |

---

## Stack Tecnológico

### Dependencias Obligatorias (pom.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.6</version>
        <relativePath/>
    </parent>
    
    <groupId>{{GROUP_ID}}</groupId>
    <artifactId>{{ARTIFACT_ID}}</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>{{PROJECT_NAME}}</name>
    <description>API REST - {{PROJECT_DESCRIPTION}}</description>
    
    <properties>
        <java.version>21</java.version>
        <mapstruct.version>1.6.3</mapstruct.version>
        <jjwt.version>0.13.0</jjwt.version>
        <springdoc.version>2.8.13</springdoc.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        
        <!-- Base de Datos -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>test</scope>
        </dependency>
        
        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        
        <!-- MapStruct -->
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- OpenAPI / Swagger -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>${springdoc.version}</version>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${mapstruct.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok-mapstruct-binding</artifactId>
                            <version>0.2.0</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## Patrones de Código

### Plantilla de Entidad JPA

```java
package {{PACKAGE}}.domain;

import jakarta.persistence.*;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.ToString;

import java.time.LocalDate;
import java.util.HashSet;
import java.util.Set;

@Data
@Entity
@Table(name = "{{TABLE_NAME}}")
public class {{EntityName}} {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private LocalDate date;

    // Relación ManyToOne - FetchType.LAZY obligatorio
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "{{foreign_key}}_id", nullable = false)
    private {{RelatedEntity}} {{relatedEntity}};

    // Relación ManyToMany - Usar Set, excluir de ToString/EqualsAndHashCode
    @ManyToMany(fetch = FetchType.LAZY, cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
            name = "{{join_table_name}}",
            joinColumns = @JoinColumn(name = "{{entity}}_id"),
            inverseJoinColumns = @JoinColumn(name = "{{related}}_id")
    )
    @ToString.Exclude
    @EqualsAndHashCode.Exclude
    private Set<{{RelatedEntity}}> {{relatedEntities}} = new HashSet<>();

    // Métodos helper para relaciones bidireccionales
    public void add{{RelatedEntity}}({{RelatedEntity}} {{related}}) {
        this.{{relatedEntities}}.add({{related}});
        {{related}}.get{{EntityName}}s().add(this);
    }

    public void remove{{RelatedEntity}}({{RelatedEntity}} {{related}}) {
        this.{{relatedEntities}}.remove({{related}});
        {{related}}.get{{EntityName}}s().remove(this);
    }
}
```

### Plantilla de Interfaz de Servicio

```java
package {{PACKAGE}}.service;

import {{PACKAGE}}.domain.{{EntityName}};
import {{PACKAGE}}.dto.{{EntityName}}RequestDto;
import {{PACKAGE}}.dto.{{EntityName}}ResponseDto;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

public interface I{{EntityName}}Service {

    Page<{{EntityName}}ResponseDto> findAll(String name, Pageable pageable);

    {{EntityName}} findById(Long id);

    {{EntityName}} save({{EntityName}}RequestDto requestDto);

    {{EntityName}} update(Long id, {{EntityName}}RequestDto requestDto);

    void deleteById(Long id);
}
```

### Plantilla de Implementación de Servicio

```java
package {{PACKAGE}}.service;

import {{PACKAGE}}.domain.{{EntityName}};
import {{PACKAGE}}.dto.{{EntityName}}RequestDto;
import {{PACKAGE}}.dto.{{EntityName}}ResponseDto;
import {{PACKAGE}}.exception.ResourceNotFoundException;
import {{PACKAGE}}.mapper.{{EntityName}}Mapper;
import {{PACKAGE}}.repository.{{EntityName}}Repository;
import lombok.RequiredArgsConstructor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class {{EntityName}}Service implements I{{EntityName}}Service {

    private static final Logger logger = LoggerFactory.getLogger({{EntityName}}Service.class);

    private final {{EntityName}}Repository {{entityName}}Repository;
    private final {{EntityName}}Mapper {{entityName}}Mapper;

    @Override
    @Transactional(readOnly = true)
    public Page<{{EntityName}}ResponseDto> findAll(String name, Pageable pageable) {
        logger.debug("Buscando {{entityName}}s en el servicio (name: '{}', pageable: {}).", name, pageable);
        
        Page<{{EntityName}}> {{entityName}}sPage;
        if (name != null && !name.isBlank()) {
            {{entityName}}sPage = {{entityName}}Repository.findByNameContainingIgnoreCase(name, pageable);
        } else {
            {{entityName}}sPage = {{entityName}}Repository.findAll(pageable);
        }
        
        Page<{{EntityName}}ResponseDto> responsePage = {{entityName}}sPage.map({{entityName}}Mapper::toResponseDto);
        logger.info("Encontrados {} {{entityName}}s paginados y mapeados a DTOs.", {{entityName}}sPage.getTotalElements());
        return responsePage;
    }

    @Override
    @Transactional(readOnly = true)
    public {{EntityName}} findById(Long id) {
        logger.debug("Buscando {{entityName}} con ID: {}", id);
        return {{entityName}}Repository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("{{EntityName}} no encontrado con id: " + id));
    }

    @Override
    @Transactional
    public {{EntityName}} save({{EntityName}}RequestDto requestDto) {
        logger.debug("Procesando save de {{entityName}}: {}", requestDto.getName());
        
        {{EntityName}} {{entityName}} = {{entityName}}Mapper.toEntity(requestDto);
        
        // TODO: Cargar y asignar relaciones (categorías, etc.)
        
        {{EntityName}} saved{{EntityName}} = {{entityName}}Repository.save({{entityName}});
        logger.info("{{EntityName}} '{}' guardado con ID: {}", saved{{EntityName}}.getName(), saved{{EntityName}}.getId());
        return saved{{EntityName}};
    }

    @Override
    @Transactional
    public {{EntityName}} update(Long id, {{EntityName}}RequestDto requestDto) {
        logger.debug("Actualizando {{entityName}} con ID: {}", id);
        
        {{EntityName}} existing{{EntityName}} = findById(id);
        {{entityName}}Mapper.updateEntityFromDto(requestDto, existing{{EntityName}});
        
        // TODO: Actualizar relaciones si es necesario
        
        {{EntityName}} updated{{EntityName}} = {{entityName}}Repository.save(existing{{EntityName}});
        logger.info("{{EntityName}} '{}' actualizado con ID: {}", updated{{EntityName}}.getName(), updated{{EntityName}}.getId());
        return updated{{EntityName}};
    }

    @Override
    @Transactional
    public void deleteById(Long id) {
        logger.debug("Eliminando {{entityName}} con ID: {}", id);
        
        {{EntityName}} {{entityName}} = findById(id);
        {{entityName}}Repository.delete({{entityName}});
        logger.info("{{EntityName}} con ID: {} eliminado exitosamente.", id);
    }
}
```

### Plantilla de Controlador REST

```java
package {{PACKAGE}}.controller;

import {{PACKAGE}}.dto.{{EntityName}}RequestDto;
import {{PACKAGE}}.dto.{{EntityName}}ResponseDto;
import {{PACKAGE}}.mapper.{{EntityName}}Mapper;
import {{PACKAGE}}.service.I{{EntityName}}Service;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/{{entity-path}}")
@RequiredArgsConstructor
@Tag(name = "{{EntityNamePlural}}", description = "Operaciones relacionadas con la gestión de {{entityNamePlural}}")
public class {{EntityName}}Controller {

    private static final Logger logger = LoggerFactory.getLogger({{EntityName}}Controller.class);

    private final I{{EntityName}}Service {{entityName}}Service;
    private final {{EntityName}}Mapper {{entityName}}Mapper;

    @GetMapping
    @PreAuthorize("hasAnyRole('ADMIN', 'USER')")
    @Operation(summary = "Obtener todos los {{entityNamePlural}} paginados",
            description = "Retorna una lista paginada de {{entityNamePlural}}. Permite filtrar por nombre.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "Lista de {{entityNamePlural}} obtenida exitosamente"),
            @ApiResponse(responseCode = "403", description = "Acceso denegado")
    })
    public ResponseEntity<Page<{{EntityName}}ResponseDto>> getAll{{EntityNamePlural}}(
            @RequestParam(required = false) String name,
            @PageableDefault(page = 0, size = 10, sort = "name") Pageable pageable) {
        logger.info("Recibida solicitud GET /{{entity-path}} con nombre '{}' y paginación {}.", name, pageable);
        Page<{{EntityName}}ResponseDto> {{entityNamePlural}} = {{entityName}}Service.findAll(name, pageable);
        return ResponseEntity.ok({{entityNamePlural}});
    }

    @GetMapping("/{id}")
    @PreAuthorize("hasAnyRole('ADMIN', 'USER')")
    @Operation(summary = "Obtener un {{entityName}} por su ID",
            description = "Retorna los detalles de un {{entityName}} específico por su ID.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "{{EntityName}} encontrado exitosamente"),
            @ApiResponse(responseCode = "404", description = "{{EntityName}} no encontrado"),
            @ApiResponse(responseCode = "403", description = "Acceso denegado")
    })
    public ResponseEntity<{{EntityName}}ResponseDto> get{{EntityName}}ById(@PathVariable Long id) {
        logger.info("Recibida solicitud GET /{{entity-path}}/{}", id);
        {{EntityName}}ResponseDto responseDto = {{entityName}}Mapper.toResponseDto({{entityName}}Service.findById(id));
        return ResponseEntity.ok(responseDto);
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    @Operation(summary = "Crear un nuevo {{entityName}}",
            description = "Permite a un administrador crear un nuevo {{entityName}} en el sistema.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "201", description = "{{EntityName}} creado exitosamente"),
            @ApiResponse(responseCode = "400", description = "Solicitud inválida"),
            @ApiResponse(responseCode = "403", description = "Acceso denegado")
    })
    public ResponseEntity<{{EntityName}}ResponseDto> create{{EntityName}}(@Valid @RequestBody {{EntityName}}RequestDto requestDto) {
        logger.info("Recibida solicitud POST para crear {{entityName}}: {}", requestDto.getName());
        {{EntityName}}ResponseDto responseDto = {{entityName}}Mapper.toResponseDto({{entityName}}Service.save(requestDto));
        logger.debug("{{EntityName}} creado exitosamente con ID: {}", responseDto.getId());
        return new ResponseEntity<>(responseDto, HttpStatus.CREATED);
    }

    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    @Operation(summary = "Actualizar un {{entityName}} existente",
            description = "Permite a un administrador actualizar un {{entityName}} existente.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "{{EntityName}} actualizado exitosamente"),
            @ApiResponse(responseCode = "400", description = "Solicitud inválida"),
            @ApiResponse(responseCode = "404", description = "{{EntityName}} no encontrado"),
            @ApiResponse(responseCode = "403", description = "Acceso denegado")
    })
    public ResponseEntity<{{EntityName}}ResponseDto> update{{EntityName}}(
            @PathVariable Long id,
            @Valid @RequestBody {{EntityName}}RequestDto requestDto) {
        logger.info("Recibida solicitud PUT para actualizar {{entityName}} con ID: {}", id);
        {{EntityName}}ResponseDto responseDto = {{entityName}}Mapper.toResponseDto({{entityName}}Service.update(id, requestDto));
        return ResponseEntity.ok(responseDto);
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    @Operation(summary = "Eliminar un {{entityName}}",
            description = "Permite a un administrador eliminar un {{entityName}} del sistema.")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "204", description = "{{EntityName}} eliminado exitosamente"),
            @ApiResponse(responseCode = "404", description = "{{EntityName}} no encontrado"),
            @ApiResponse(responseCode = "403", description = "Acceso denegado")
    })
    public ResponseEntity<Void> delete{{EntityName}}(@PathVariable Long id) {
        logger.info("Recibida solicitud DELETE para {{entityName}} con ID: {}", id);
        {{entityName}}Service.deleteById(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Plantilla de Repositorio

```java
package {{PACKAGE}}.repository;

import {{PACKAGE}}.domain.{{EntityName}};
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.lang.NonNull;

import java.util.List;
import java.util.Optional;

public interface {{EntityName}}Repository extends JpaRepository<{{EntityName}}, Long> {

    // Query method para búsqueda por nombre
    Page<{{EntityName}}> findByNameContainingIgnoreCase(String name, Pageable pageable);

    // JOIN FETCH para cargar relaciones específicas
    @Query("SELECT e FROM {{EntityName}} e JOIN FETCH e.{{relation}} LEFT JOIN FETCH e.{{relations}}")
    List<{{EntityName}}> findAllWith{{Relation}}And{{Relations}}();

    // JOIN FETCH con filtro por ID
    @Query("SELECT e FROM {{EntityName}} e JOIN FETCH e.{{relation}} LEFT JOIN FETCH e.{{relations}} WHERE e.id = :id")
    Optional<{{EntityName}}> findByIdWith{{Relation}}And{{Relations}}(Long id);

    // EntityGraph para múltiples relaciones
    @EntityGraph(attributePaths = {"{{relation}}", "{{relations}}"})
    @Query("SELECT e FROM {{EntityName}} e")
    List<{{EntityName}}> findAllWithAllDetails();

    // Override de métodos base con EntityGraph
    @Override
    @NonNull
    @EntityGraph(attributePaths = {"{{relation}}"})
    List<{{EntityName}}> findAll();

    @Override
    @NonNull
    @EntityGraph(attributePaths = {"{{relation}}"})
    Optional<{{EntityName}}> findById(@NonNull Long id);
}
```

### Plantilla de DTO de Entrada

```java
package {{PACKAGE}}.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.Data;

import java.time.LocalDate;
import java.util.Set;

@Data
@Schema(description = "Detalles de la solicitud para crear/actualizar un {{entityName}}")
public class {{EntityName}}RequestDto {

    @Schema(description = "Nombre del {{entityName}}", example = "Ejemplo de {{EntityName}}")
    @NotBlank(message = "El nombre del {{entityName}} no puede estar vacío.")
    private String name;

    @Schema(description = "Fecha del {{entityName}}", example = "2024-12-01")
    @NotNull(message = "La fecha no puede ser nula.")
    private LocalDate date;

    @Schema(description = "Ubicación del {{entityName}}", example = "Ciudad, País")
    @NotBlank(message = "La ubicación no puede estar vacía.")
    private String location;

    @Schema(description = "ID de la categoría", example = "1")
    @NotNull(message = "La categoría es obligatoria.")
    private Long categoryId;

    @Schema(description = "IDs de los elementos relacionados")
    private Set<Long> relatedIds;
}
```

### Plantilla de DTO de Salida

```java
package {{PACKAGE}}.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

import java.time.LocalDate;
import java.util.Set;

@Data
@Schema(description = "Respuesta con los detalles de un {{entityName}}")
public class {{EntityName}}ResponseDto {

    @Schema(description = "ID del {{entityName}}")
    private Long id;

    @Schema(description = "Nombre del {{entityName}}")
    private String name;

    @Schema(description = "Fecha del {{entityName}}")
    private LocalDate date;

    @Schema(description = "Ubicación del {{entityName}}")
    private String location;

    @Schema(description = "ID de la categoría")
    private Long categoryId;

    @Schema(description = "Nombre de la categoría")
    private String categoryName;

    @Schema(description = "Elementos relacionados")
    private Set<{{Related}}ResponseDto> {{related}};
}
```

### Plantilla de Mapper MapStruct

```java
package {{PACKAGE}}.mapper;

import {{PACKAGE}}.domain.{{EntityName}};
import {{PACKAGE}}.dto.{{EntityName}}RequestDto;
import {{PACKAGE}}.dto.{{EntityName}}ResponseDto;
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.MappingTarget;

@Mapper(componentModel = "spring")
public interface {{EntityName}}Mapper {

    @Mapping(target = "id", ignore = true)
    @Mapping(target = "{{relation}}", ignore = true)
    @Mapping(target = "{{relations}}", ignore = true)
    {{EntityName}} toEntity({{EntityName}}RequestDto requestDto);

    @Mapping(target = "{{relation}}Id", source = "{{relation}}.id")
    @Mapping(target = "{{relation}}Name", source = "{{relation}}.name")
    {{EntityName}}ResponseDto toResponseDto({{EntityName}} {{entityName}});

    @Mapping(target = "id", ignore = true)
    @Mapping(target = "{{relation}}", ignore = true)
    @Mapping(target = "{{relations}}", ignore = true)
    void updateEntityFromDto({{EntityName}}RequestDto dto, @MappingTarget {{EntityName}} {{entityName}});
}
```

### Plantilla de Excepción Personalizada

```java
package {{PACKAGE}}.exception;

public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

### Plantilla de GlobalExceptionHandler

```java
package {{PACKAGE}}.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<Object> handleResourceNotFoundException(ResourceNotFoundException ex) {
        Map<String, Object> body = new HashMap<>();
        body.put("status", HttpStatus.NOT_FOUND.value());
        body.put("error", "Not Found");
        body.put("message", ex.getMessage());
        return new ResponseEntity<>(body, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Object> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage())
        );

        Map<String, Object> body = new HashMap<>();
        body.put("status", HttpStatus.BAD_REQUEST.value());
        body.put("error", "Bad Request");
        body.put("message", "Errores de validación");
        body.put("errors", errors);
        return new ResponseEntity<>(body, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, String>> handleGeneralException(Exception ex) {
        Map<String, String> errorDetails = new HashMap<>();
        errorDetails.put("error", "Error Interno del Servidor");
        errorDetails.put("message", "Ocurrió un error inesperado. Por favor, inténtalo de nuevo más tarde.");
        return new ResponseEntity<>(errorDetails, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

---

## Plantillas de Código

Para plantillas completas adicionales, ver archivos de referencia:
- [references/security-templates.md](references/security-templates.md) - Módulo completo de seguridad JWT
- [references/config-templates.md](references/config-templates.md) - Archivos de configuración (properties, logback)
- [references/testing-templates.md](references/testing-templates.md) - Plantillas de tests con JUnit 5 y Mockito
- [references/docker-templates.md](references/docker-templates.md) - Dockerfile y docker-compose

---

## Generación del README.md

El proyecto generado **DEBE incluir un archivo README.md** completo con la siguiente estructura:

### Plantilla README.md

```markdown
# {{PROJECT_NAME}}

API REST para {{PROJECT_DESCRIPTION}}.

## Tabla de Contenidos

- [Descripción](#descripción)
- [Stack Tecnológico](#stack-tecnológico)
- [Arquitectura](#arquitectura)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Configuración](#configuración)
- [Ejecución](#ejecución)
- [Endpoints API](#endpoints-api)
- [Autenticación](#autenticación)
- [Convenciones de Código](#convenciones-de-código)
- [Testing](#testing)
- [Despliegue](#despliegue)

## Descripción

{{PROJECT_DESCRIPTION}}

## Stack Tecnológico

| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| Java | 21 | Lenguaje principal |
| Spring Boot | 3.5.6 | Framework de aplicación |
| Spring Data JPA | - | Capa de persistencia |
| Spring Security | - | Autenticación y autorización |
| PostgreSQL | 16+ | Base de datos (dev/prod) |
| H2 | - | Base de datos (testing) |
| MapStruct | 1.6.3 | Mapeo entidad-DTO |
| JWT (jjwt) | 0.13.0 | Autenticación stateless |
| SpringDoc OpenAPI | 2.8.13 | Documentación API |
| Lombok | - | Reducción de boilerplate |
| JUnit 5 | - | Framework de testing |
| Mockito | - | Mocking de dependencias |

## Arquitectura

El proyecto sigue una **arquitectura en capas tradicional** de Spring Boot:

```
┌─────────────────────────────────────────────────────────────┐
│                    Capa de Presentación                      │
│                    (controllers/)                            │
│  - Maneja requests/responses HTTP                           │
│  - Documentación OpenAPI/Swagger                            │
│  - Control de acceso con @PreAuthorize                      │
├─────────────────────────────────────────────────────────────┤
│                    Capa de Servicio                          │
│                    (service/)                                │
│  - Lógica de negocio                                        │
│  - Transacciones (@Transactional)                           │
│  - Orquestación de operaciones                              │
├─────────────────────────────────────────────────────────────┤
│                    Capa de Dominio                           │
│                    (domain/)                                 │
│  - Entidades JPA                                            │
│  - Integridad del modelo de datos                           │
├─────────────────────────────────────────────────────────────┤
│                    Capa de Infraestructura                   │
│         (repository/, security/, mapper/)                    │
│  - Repositorios Spring Data JPA                             │
│  - Configuración de seguridad                               │
│  - Mappers MapStruct                                        │
└─────────────────────────────────────────────────────────────┘
```

### Principios de Diseño

- **Arquitectura en Capas**: Separación clara de responsabilidades
- **SOLID**: Principios aplicados en toda la aplicación
- **DRY**: Centralización con MapStruct para mapeos
- **Inyección de Dependencias**: Via constructor con `@RequiredArgsConstructor`

## Estructura del Proyecto

```
src/
├── main/java/{{PACKAGE_PATH}}/
│   ├── {{ProjectName}}Application.java
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── domain/
│   ├── dto/
│   ├── mapper/
│   ├── exception/
│   ├── data/
│   └── security/
└── test/java/{{PACKAGE_PATH}}/
```

## Configuración

### Variables de Entorno Requeridas

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `NEON_DB_URL` | URL de conexión PostgreSQL | `jdbc:postgresql://...` |
| `NEON_DB_USER` | Usuario de base de datos | `admin` |
| `NEON_DB_PASSWORD` | Contraseña de base de datos | `****` |
| `JWT_SECRET` | Secret para JWT (mín. 64 chars) | `your-secret-key...` |
| `JWT_EXPIRATION` | Expiración del token (ms) | `86400000` |

### Perfiles de Configuración

| Perfil | Base de Datos | Logging | Swagger |
|--------|---------------|---------|---------|
| `dev` | PostgreSQL (Neon) | DEBUG | Público |
| `prod` | PostgreSQL (Neon) | INFO | Protegido |
| `test` | H2 (memoria) | DEBUG | N/A |

## Ejecución

### Desarrollo

```bash
# Ejecutar con perfil dev
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# Ejecutar tests
./mvnw test

# Build
./mvnw clean package
```

### Docker

```bash
# Build imagen
docker build -t {{project-name}} .

# Ejecutar con docker-compose
docker-compose up -d
```

## Endpoints API

### Base URL: `/api/v1`

| Método | Endpoint | Descripción | Roles |
|--------|----------|-------------|-------|
| POST | `/auth/login` | Autenticación | Público |
| POST | `/auth/register` | Registro de usuario | Público |
| GET | `/{{entities}}` | Listar (paginado) | ADMIN, USER |
| GET | `/{{entities}}/{id}` | Obtener por ID | ADMIN, USER |
| POST | `/{{entities}}` | Crear | ADMIN |
| PUT | `/{{entities}}/{id}` | Actualizar | ADMIN |
| DELETE | `/{{entities}}/{id}` | Eliminar | ADMIN |

### Swagger UI

Disponible en: `http://localhost:8080/swagger-ui.html` (solo en perfil `dev`)

## Autenticación

El sistema usa **JWT (JSON Web Tokens)** para autenticación stateless.

### Flujo de Autenticación

1. POST `/api/v1/auth/login` con credenciales
2. Recibir token JWT en respuesta
3. Incluir token en header: `Authorization: Bearer <token>`

### Roles

| Rol | Permisos |
|-----|----------|
| `ROLE_ADMIN` | CRUD completo |
| `ROLE_USER` | Solo lectura |

## Convenciones de Código

### Idioma

| Elemento | Idioma |
|----------|--------|
| Código (clases, métodos, variables) | Inglés |
| Documentación, comentarios, logs | Español |
| Mensajes de validación | Español |
| Mensajes de error | Español |

### Nomenclatura

| Tipo | Convención | Ejemplo |
|------|------------|---------|
| Clases | PascalCase | `EventService` |
| Interfaces Servicio | `I` + PascalCase | `IEventService` |
| Variables/Métodos | camelCase | `findById` |
| Constantes | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE` |
| Tablas BD | snake_case plural | `events` |

### Anotaciones Lombok

```java
// Entidades
@Data
@Entity
@ToString.Exclude    // En colecciones
@EqualsAndHashCode.Exclude  // En colecciones

// Servicios y Controllers
@RequiredArgsConstructor
```

### Logging

```java
private static final Logger logger = LoggerFactory.getLogger(ClassName.class);

// Niveles
logger.debug("Detalles técnicos");    // DEBUG
logger.info("Operaciones principales"); // INFO
logger.warn("Situaciones anómalas");   // WARN
logger.error("Errores", ex);           // ERROR
```

## Testing

### Estructura

- Tests unitarios: `src/test/java/.../service/`
- Tests de integración: `src/test/java/.../controller/`

### Convenciones

- Patrón AAA: Arrange-Act-Assert (Given-When-Then)
- Naming: `should[Result]When[Condition]`
- Usar `@DisplayName` para descripciones en español

### Ejecutar Tests

```bash
./mvnw test                        # Todos los tests
./mvnw test -Dtest=EventServiceTest # Test específico
./mvnw verify                      # Tests + verificación
```

## Despliegue

### Docker

```dockerfile
FROM eclipse-temurin:21-jdk as build
# ... build stage

FROM eclipse-temurin:21-jre
# ... runtime stage
EXPOSE 8080
```

### Verificación de Salud

```bash
curl http://localhost:8080/actuator/health
```

---

Generado con Spring Boot API Scaffold Generator
```

---

## Validación Final

### Checklist de Scaffold Completo

```markdown
## Configuración
[ ] pom.xml con todas las dependencias y plugins
[ ] application.properties base
[ ] application-dev.properties con variables de entorno
[ ] application-prod.properties con configuración producción
[ ] application-test.properties para H2
[ ] logback-spring.xml configurado
[ ] README.md completo con guía de desarrollo

## Estructura de Capas
[ ] controller/ con endpoints REST documentados
[ ] service/ con interfaces e implementaciones
[ ] repository/ con queries optimizadas
[ ] domain/ con entidades JPA
[ ] dto/ con Request y Response DTOs
[ ] mapper/ con MapStruct mappers
[ ] exception/ con GlobalExceptionHandler

## Seguridad
[ ] security/config/SecurityConfig.java
[ ] security/config/OpenApiConfig.java
[ ] security/controller/AuthController.java
[ ] security/dto/ con LoginDto, RegisterDto, JwtAuthResponseDto
[ ] security/jwt/ con Generator, Filter, EntryPoint
[ ] security/service/UserDetailsServiceImpl.java
[ ] domain/User.java y domain/Role.java

## Entidades y Relaciones
[ ] FetchType.LAZY en todas las relaciones
[ ] @ToString.Exclude en colecciones
[ ] @EqualsAndHashCode.Exclude en colecciones
[ ] Métodos helper para relaciones bidireccionales
[ ] Set<> para colecciones ManyToMany

## Servicios
[ ] Interfaces I*Service definidas
[ ] Implementaciones con @Service
[ ] @Transactional en métodos de escritura
[ ] @Transactional(readOnly = true) en consultas
[ ] Logger con mensajes en español

## Controladores
[ ] Documentación OpenAPI completa
[ ] @PreAuthorize para control de acceso
[ ] @Valid en DTOs de entrada
[ ] Paginación con Pageable
[ ] Logging de entrada/salida

## Testing
[ ] Estructura espejo en src/test/java
[ ] Tests de servicio con Mockito
[ ] Tests de controller con MockMvc
[ ] @DisplayName en español
```

### Comandos de Verificación

```bash
# Verificar que compila
./mvnw clean compile

# Verificar tests
./mvnw test

# Build completo
./mvnw clean package

# Ejecutar aplicación
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev
```

---

## Instrucciones de Ejecución para el Agente

1. **PREGUNTAR** nombre del proyecto y Group ID al usuario (OBLIGATORIO)
2. **LEER** completamente el documento de arquitectura provisto
3. **IDENTIFICAR** entidades, relaciones, endpoints y roles
4. **CREAR** estructura base de carpetas según arquitectura en capas
5. **GENERAR** `pom.xml` con todas las dependencias
6. **CREAR** archivos de configuración (properties, logback)
7. **IMPLEMENTAR** módulo de seguridad JWT completo
8. **GENERAR** entidades JPA con relaciones y métodos helper
9. **CREAR** repositorios con queries optimizadas
10. **IMPLEMENTAR** servicios con interfaces e implementaciones
11. **GENERAR** controladores con documentación OpenAPI
12. **CREAR** DTOs de entrada/salida con validaciones
13. **IMPLEMENTAR** mappers MapStruct
14. **CREAR** GlobalExceptionHandler y excepciones
15. **GENERAR** estructura de tests
16. **GENERAR** README.md usando la plantilla
17. **VALIDAR** que el proyecto compila sin errores

### Notas Importantes

- **SIEMPRE** usar FetchType.LAZY en relaciones JPA
- **NUNCA** exponer entidades directamente en controllers
- **SIEMPRE** usar DTOs para entrada/salida
- **CÓDIGO** en inglés, **mensajes/logs/documentación** en español
- **INCLUIR** TODOs donde se requiera lógica de negocio específica
- **RESPETAR** convenciones de nomenclatura estrictamente
- **MANTENER** consistencia con los patrones definidos
