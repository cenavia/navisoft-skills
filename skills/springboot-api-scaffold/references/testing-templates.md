# Plantillas de Testing - Spring Boot

Este documento contiene las plantillas de tests con JUnit 5 y Mockito.

## Tabla de Contenidos

1. [Tests de Servicio](#tests-de-servicio)
2. [Tests de Controlador](#tests-de-controlador)
3. [Tests de Integración](#tests-de-integración)
4. [Fixtures y Test Data](#fixtures-y-test-data)

---

## Tests de Servicio

### Plantilla Base de Test de Servicio

```java
package {{PACKAGE}}.service;

import {{PACKAGE}}.domain.{{EntityName}};
import {{PACKAGE}}.dto.{{EntityName}}RequestDto;
import {{PACKAGE}}.dto.{{EntityName}}ResponseDto;
import {{PACKAGE}}.exception.ResourceNotFoundException;
import {{PACKAGE}}.mapper.{{EntityName}}Mapper;
import {{PACKAGE}}.repository.{{EntityName}}Repository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyLong;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
@DisplayName("Tests de {{EntityName}}Service")
class {{EntityName}}ServiceTest {

    @Mock
    private {{EntityName}}Repository {{entityName}}Repository;

    @Mock
    private {{EntityName}}Mapper {{entityName}}Mapper;

    @InjectMocks
    private {{EntityName}}Service {{entityName}}Service;

    private {{EntityName}} {{entityName}};
    private {{EntityName}}RequestDto requestDto;
    private {{EntityName}}ResponseDto responseDto;

    @BeforeEach
    void setUp() {
        // Preparar datos de prueba
        {{entityName}} = new {{EntityName}}();
        {{entityName}}.setId(1L);
        {{entityName}}.setName("Test {{EntityName}}");
        {{entityName}}.setDate(LocalDate.now());

        requestDto = new {{EntityName}}RequestDto();
        requestDto.setName("Test {{EntityName}}");
        requestDto.setDate(LocalDate.now());

        responseDto = new {{EntityName}}ResponseDto();
        responseDto.setId(1L);
        responseDto.setName("Test {{EntityName}}");
        responseDto.setDate(LocalDate.now());
    }

    @Nested
    @DisplayName("Tests de findById")
    class FindByIdTests {

        @Test
        @DisplayName("Debe retornar {{EntityName}} cuando el ID existe")
        void shouldReturn{{EntityName}}WhenIdExists() {
            // Given
            when({{entityName}}Repository.findById(1L)).thenReturn(Optional.of({{entityName}}));

            // When
            {{EntityName}} found = {{entityName}}Service.findById(1L);

            // Then
            assertNotNull(found);
            assertEquals(1L, found.getId());
            assertEquals("Test {{EntityName}}", found.getName());
            verify({{entityName}}Repository, times(1)).findById(1L);
        }

        @Test
        @DisplayName("Debe lanzar ResourceNotFoundException cuando el ID no existe")
        void shouldThrowResourceNotFoundExceptionWhenIdDoesNotExist() {
            // Given
            when({{entityName}}Repository.findById(anyLong())).thenReturn(Optional.empty());

            // When & Then
            ResourceNotFoundException thrown = assertThrows(
                    ResourceNotFoundException.class,
                    () -> {{entityName}}Service.findById(99L)
            );

            assertEquals("{{EntityName}} no encontrado con id: 99", thrown.getMessage());
            verify({{entityName}}Repository, times(1)).findById(99L);
        }
    }

    @Nested
    @DisplayName("Tests de findAll")
    class FindAllTests {

        @Test
        @DisplayName("Debe retornar página de {{EntityName}}s")
        void shouldReturnPageOf{{EntityName}}s() {
            // Given
            Pageable pageable = PageRequest.of(0, 10);
            Page<{{EntityName}}> {{entityName}}Page = new PageImpl<>(List.of({{entityName}}));

            when({{entityName}}Repository.findAll(pageable)).thenReturn({{entityName}}Page);
            when({{entityName}}Mapper.toResponseDto(any({{EntityName}}.class))).thenReturn(responseDto);

            // When
            Page<{{EntityName}}ResponseDto> result = {{entityName}}Service.findAll(null, pageable);

            // Then
            assertNotNull(result);
            assertEquals(1, result.getTotalElements());
            verify({{entityName}}Repository, times(1)).findAll(pageable);
        }

        @Test
        @DisplayName("Debe filtrar por nombre cuando se proporciona")
        void shouldFilterByNameWhenProvided() {
            // Given
            String searchName = "Test";
            Pageable pageable = PageRequest.of(0, 10);
            Page<{{EntityName}}> {{entityName}}Page = new PageImpl<>(List.of({{entityName}}));

            when({{entityName}}Repository.findByNameContainingIgnoreCase(searchName, pageable))
                    .thenReturn({{entityName}}Page);
            when({{entityName}}Mapper.toResponseDto(any({{EntityName}}.class))).thenReturn(responseDto);

            // When
            Page<{{EntityName}}ResponseDto> result = {{entityName}}Service.findAll(searchName, pageable);

            // Then
            assertNotNull(result);
            verify({{entityName}}Repository, times(1)).findByNameContainingIgnoreCase(searchName, pageable);
            verify({{entityName}}Repository, never()).findAll(pageable);
        }
    }

    @Nested
    @DisplayName("Tests de save")
    class SaveTests {

        @Test
        @DisplayName("Debe guardar {{EntityName}} exitosamente")
        void shouldSave{{EntityName}}Successfully() {
            // Given
            {{EntityName}} {{entityName}}ToSave = new {{EntityName}}();
            {{entityName}}ToSave.setName("Test {{EntityName}}");

            when({{entityName}}Mapper.toEntity(requestDto)).thenReturn({{entityName}}ToSave);
            when({{entityName}}Repository.save(any({{EntityName}}.class))).thenAnswer(invocation -> {
                {{EntityName}} saved = invocation.getArgument(0);
                saved.setId(1L);
                return saved;
            });

            // When
            {{EntityName}} saved = {{entityName}}Service.save(requestDto);

            // Then
            assertNotNull(saved);
            assertEquals(1L, saved.getId());
            verify({{entityName}}Mapper, times(1)).toEntity(requestDto);
            verify({{entityName}}Repository, times(1)).save(any({{EntityName}}.class));
        }
    }

    @Nested
    @DisplayName("Tests de update")
    class UpdateTests {

        @Test
        @DisplayName("Debe actualizar {{EntityName}} existente")
        void shouldUpdate{{EntityName}}Successfully() {
            // Given
            when({{entityName}}Repository.findById(1L)).thenReturn(Optional.of({{entityName}}));
            doNothing().when({{entityName}}Mapper).updateEntityFromDto(requestDto, {{entityName}});
            when({{entityName}}Repository.save({{entityName}})).thenReturn({{entityName}});

            // When
            {{EntityName}} updated = {{entityName}}Service.update(1L, requestDto);

            // Then
            assertNotNull(updated);
            verify({{entityName}}Repository, times(1)).findById(1L);
            verify({{entityName}}Mapper, times(1)).updateEntityFromDto(requestDto, {{entityName}});
            verify({{entityName}}Repository, times(1)).save({{entityName}});
        }

        @Test
        @DisplayName("Debe lanzar excepción al actualizar {{EntityName}} inexistente")
        void shouldThrowExceptionWhenUpdatingNonExistent{{EntityName}}() {
            // Given
            when({{entityName}}Repository.findById(anyLong())).thenReturn(Optional.empty());

            // When & Then
            assertThrows(
                    ResourceNotFoundException.class,
                    () -> {{entityName}}Service.update(99L, requestDto)
            );
            verify({{entityName}}Repository, times(1)).findById(99L);
            verify({{entityName}}Repository, never()).save(any());
        }
    }

    @Nested
    @DisplayName("Tests de deleteById")
    class DeleteByIdTests {

        @Test
        @DisplayName("Debe eliminar {{EntityName}} existente")
        void shouldDelete{{EntityName}}Successfully() {
            // Given
            when({{entityName}}Repository.findById(1L)).thenReturn(Optional.of({{entityName}}));
            doNothing().when({{entityName}}Repository).delete({{entityName}});

            // When
            {{entityName}}Service.deleteById(1L);

            // Then
            verify({{entityName}}Repository, times(1)).findById(1L);
            verify({{entityName}}Repository, times(1)).delete({{entityName}});
        }

        @Test
        @DisplayName("Debe lanzar excepción al eliminar {{EntityName}} inexistente")
        void shouldThrowExceptionWhenDeletingNonExistent{{EntityName}}() {
            // Given
            when({{entityName}}Repository.findById(anyLong())).thenReturn(Optional.empty());

            // When & Then
            assertThrows(
                    ResourceNotFoundException.class,
                    () -> {{entityName}}Service.deleteById(99L)
            );
            verify({{entityName}}Repository, times(1)).findById(99L);
            verify({{entityName}}Repository, never()).delete(any());
        }
    }
}
```

---

## Tests de Controlador

### Plantilla Base de Test de Controlador

```java
package {{PACKAGE}}.controller;

import {{PACKAGE}}.domain.{{EntityName}};
import {{PACKAGE}}.dto.{{EntityName}}RequestDto;
import {{PACKAGE}}.dto.{{EntityName}}ResponseDto;
import {{PACKAGE}}.exception.GlobalExceptionHandler;
import {{PACKAGE}}.exception.ResourceNotFoundException;
import {{PACKAGE}}.mapper.{{EntityName}}Mapper;
import {{PACKAGE}}.service.I{{EntityName}}Service;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.context.annotation.Import;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.Pageable;
import org.springframework.http.MediaType;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.web.servlet.MockMvc;

import java.time.LocalDate;
import java.util.List;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyLong;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest({{EntityName}}Controller.class)
@Import(GlobalExceptionHandler.class)
@DisplayName("Tests de {{EntityName}}Controller")
class {{EntityName}}ControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean
    private I{{EntityName}}Service {{entityName}}Service;

    @MockBean
    private {{EntityName}}Mapper {{entityName}}Mapper;

    private {{EntityName}} {{entityName}};
    private {{EntityName}}RequestDto requestDto;
    private {{EntityName}}ResponseDto responseDto;

    @BeforeEach
    void setUp() {
        {{entityName}} = new {{EntityName}}();
        {{entityName}}.setId(1L);
        {{entityName}}.setName("Test {{EntityName}}");
        {{entityName}}.setDate(LocalDate.now());

        requestDto = new {{EntityName}}RequestDto();
        requestDto.setName("Test {{EntityName}}");
        requestDto.setDate(LocalDate.now());

        responseDto = new {{EntityName}}ResponseDto();
        responseDto.setId(1L);
        responseDto.setName("Test {{EntityName}}");
        responseDto.setDate(LocalDate.now());
    }

    @Nested
    @DisplayName("GET /api/v1/{{entity-path}}")
    class GetAllTests {

        @Test
        @WithMockUser(roles = "USER")
        @DisplayName("Debe retornar lista paginada de {{EntityName}}s")
        void shouldReturnPageOf{{EntityName}}s() throws Exception {
            // Given
            Page<{{EntityName}}ResponseDto> page = new PageImpl<>(List.of(responseDto));
            when({{entityName}}Service.findAll(any(), any(Pageable.class))).thenReturn(page);

            // When & Then
            mockMvc.perform(get("/api/v1/{{entity-path}}")
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.content").isArray())
                    .andExpect(jsonPath("$.content[0].id").value(1L))
                    .andExpect(jsonPath("$.content[0].name").value("Test {{EntityName}}"));

            verify({{entityName}}Service, times(1)).findAll(any(), any(Pageable.class));
        }

        @Test
        @DisplayName("Debe retornar 401 cuando no está autenticado")
        void shouldReturn401WhenNotAuthenticated() throws Exception {
            mockMvc.perform(get("/api/v1/{{entity-path}}")
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isUnauthorized());
        }
    }

    @Nested
    @DisplayName("GET /api/v1/{{entity-path}}/{id}")
    class GetByIdTests {

        @Test
        @WithMockUser(roles = "USER")
        @DisplayName("Debe retornar {{EntityName}} cuando existe")
        void shouldReturn{{EntityName}}WhenExists() throws Exception {
            // Given
            when({{entityName}}Service.findById(1L)).thenReturn({{entityName}});
            when({{entityName}}Mapper.toResponseDto({{entityName}})).thenReturn(responseDto);

            // When & Then
            mockMvc.perform(get("/api/v1/{{entity-path}}/1")
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.id").value(1L))
                    .andExpect(jsonPath("$.name").value("Test {{EntityName}}"));

            verify({{entityName}}Service, times(1)).findById(1L);
        }

        @Test
        @WithMockUser(roles = "USER")
        @DisplayName("Debe retornar 404 cuando no existe")
        void shouldReturn404WhenNotExists() throws Exception {
            // Given
            when({{entityName}}Service.findById(anyLong()))
                    .thenThrow(new ResourceNotFoundException("{{EntityName}} no encontrado con id: 99"));

            // When & Then
            mockMvc.perform(get("/api/v1/{{entity-path}}/99")
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isNotFound())
                    .andExpect(jsonPath("$.message").value("{{EntityName}} no encontrado con id: 99"));
        }
    }

    @Nested
    @DisplayName("POST /api/v1/{{entity-path}}")
    class CreateTests {

        @Test
        @WithMockUser(roles = "ADMIN")
        @DisplayName("Debe crear {{EntityName}} exitosamente")
        void shouldCreate{{EntityName}}Successfully() throws Exception {
            // Given
            when({{entityName}}Service.save(any({{EntityName}}RequestDto.class))).thenReturn({{entityName}});
            when({{entityName}}Mapper.toResponseDto({{entityName}})).thenReturn(responseDto);

            // When & Then
            mockMvc.perform(post("/api/v1/{{entity-path}}")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(requestDto)))
                    .andExpect(status().isCreated())
                    .andExpect(jsonPath("$.id").value(1L))
                    .andExpect(jsonPath("$.name").value("Test {{EntityName}}"));

            verify({{entityName}}Service, times(1)).save(any({{EntityName}}RequestDto.class));
        }

        @Test
        @WithMockUser(roles = "ADMIN")
        @DisplayName("Debe retornar 400 con datos inválidos")
        void shouldReturn400WithInvalidData() throws Exception {
            // Given
            {{EntityName}}RequestDto invalidDto = new {{EntityName}}RequestDto();
            // DTO vacío, debería fallar validación

            // When & Then
            mockMvc.perform(post("/api/v1/{{entity-path}}")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(invalidDto)))
                    .andExpect(status().isBadRequest());
        }

        @Test
        @WithMockUser(roles = "USER")
        @DisplayName("Debe retornar 403 cuando usuario no es ADMIN")
        void shouldReturn403WhenUserIsNotAdmin() throws Exception {
            // When & Then
            mockMvc.perform(post("/api/v1/{{entity-path}}")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(requestDto)))
                    .andExpect(status().isForbidden());
        }
    }

    @Nested
    @DisplayName("PUT /api/v1/{{entity-path}}/{id}")
    class UpdateTests {

        @Test
        @WithMockUser(roles = "ADMIN")
        @DisplayName("Debe actualizar {{EntityName}} exitosamente")
        void shouldUpdate{{EntityName}}Successfully() throws Exception {
            // Given
            when({{entityName}}Service.update(anyLong(), any({{EntityName}}RequestDto.class))).thenReturn({{entityName}});
            when({{entityName}}Mapper.toResponseDto({{entityName}})).thenReturn(responseDto);

            // When & Then
            mockMvc.perform(put("/api/v1/{{entity-path}}/1")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(objectMapper.writeValueAsString(requestDto)))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.id").value(1L));

            verify({{entityName}}Service, times(1)).update(eq(1L), any({{EntityName}}RequestDto.class));
        }
    }

    @Nested
    @DisplayName("DELETE /api/v1/{{entity-path}}/{id}")
    class DeleteTests {

        @Test
        @WithMockUser(roles = "ADMIN")
        @DisplayName("Debe eliminar {{EntityName}} exitosamente")
        void shouldDelete{{EntityName}}Successfully() throws Exception {
            // Given
            doNothing().when({{entityName}}Service).deleteById(1L);

            // When & Then
            mockMvc.perform(delete("/api/v1/{{entity-path}}/1")
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isNoContent());

            verify({{entityName}}Service, times(1)).deleteById(1L);
        }

        @Test
        @WithMockUser(roles = "USER")
        @DisplayName("Debe retornar 403 cuando usuario no es ADMIN")
        void shouldReturn403WhenUserIsNotAdmin() throws Exception {
            mockMvc.perform(delete("/api/v1/{{entity-path}}/1")
                            .contentType(MediaType.APPLICATION_JSON))
                    .andExpect(status().isForbidden());
        }
    }
}
```

---

## Tests de Integración

### Plantilla de Test de Integración

```java
package {{PACKAGE}}.controller;

import {{PACKAGE}}.domain.{{EntityName}};
import {{PACKAGE}}.dto.{{EntityName}}RequestDto;
import {{PACKAGE}}.repository.{{EntityName}}Repository;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
@Transactional
@DisplayName("Tests de Integración - {{EntityName}}")
class {{EntityName}}ControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private {{EntityName}}Repository {{entityName}}Repository;

    private {{EntityName}} saved{{EntityName}};

    @BeforeEach
    void setUp() {
        // Limpiar y preparar datos
        {{entityName}}Repository.deleteAll();

        {{EntityName}} {{entityName}} = new {{EntityName}}();
        {{entityName}}.setName("Test {{EntityName}}");
        {{entityName}}.setDate(LocalDate.now());
        saved{{EntityName}} = {{entityName}}Repository.save({{entityName}});
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    @DisplayName("Flujo completo CRUD de {{EntityName}}")
    void shouldPerformFullCrudFlow() throws Exception {
        // CREATE
        {{EntityName}}RequestDto createDto = new {{EntityName}}RequestDto();
        createDto.setName("Nuevo {{EntityName}}");
        createDto.setDate(LocalDate.now().plusDays(7));

        String createResponse = mockMvc.perform(post("/api/v1/{{entity-path}}")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(createDto)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.name").value("Nuevo {{EntityName}}"))
                .andReturn().getResponse().getContentAsString();

        Long createdId = objectMapper.readTree(createResponse).get("id").asLong();

        // READ
        mockMvc.perform(get("/api/v1/{{entity-path}}/" + createdId)
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(createdId))
                .andExpect(jsonPath("$.name").value("Nuevo {{EntityName}}"));

        // UPDATE
        {{EntityName}}RequestDto updateDto = new {{EntityName}}RequestDto();
        updateDto.setName("{{EntityName}} Actualizado");
        updateDto.setDate(LocalDate.now().plusDays(14));

        mockMvc.perform(put("/api/v1/{{entity-path}}/" + createdId)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(updateDto)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("{{EntityName}} Actualizado"));

        // DELETE
        mockMvc.perform(delete("/api/v1/{{entity-path}}/" + createdId)
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isNoContent());

        // VERIFY DELETED
        mockMvc.perform(get("/api/v1/{{entity-path}}/" + createdId)
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isNotFound());
    }

    @Test
    @WithMockUser(roles = "USER")
    @DisplayName("Usuario normal solo puede leer")
    void userCanOnlyRead() throws Exception {
        // GET - Permitido
        mockMvc.perform(get("/api/v1/{{entity-path}}")
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk());

        // POST - Prohibido
        {{EntityName}}RequestDto dto = new {{EntityName}}RequestDto();
        dto.setName("Test");
        dto.setDate(LocalDate.now());

        mockMvc.perform(post("/api/v1/{{entity-path}}")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(dto)))
                .andExpect(status().isForbidden());

        // PUT - Prohibido
        mockMvc.perform(put("/api/v1/{{entity-path}}/" + saved{{EntityName}}.getId())
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(dto)))
                .andExpect(status().isForbidden());

        // DELETE - Prohibido
        mockMvc.perform(delete("/api/v1/{{entity-path}}/" + saved{{EntityName}}.getId())
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isForbidden());
    }
}
```

---

## Fixtures y Test Data

### Plantilla de Fixture Builder

```java
package {{PACKAGE}}.fixtures;

import {{PACKAGE}}.domain.{{EntityName}};
import {{PACKAGE}}.dto.{{EntityName}}RequestDto;
import {{PACKAGE}}.dto.{{EntityName}}ResponseDto;

import java.time.LocalDate;

/**
 * Clase de utilidad para crear objetos de prueba de {{EntityName}}.
 * Utiliza el patrón Builder para facilitar la creación de fixtures.
 */
public class {{EntityName}}Fixture {

    private Long id = 1L;
    private String name = "Test {{EntityName}}";
    private LocalDate date = LocalDate.now();
    private String location = "Test Location";

    public static {{EntityName}}Fixture builder() {
        return new {{EntityName}}Fixture();
    }

    public {{EntityName}}Fixture withId(Long id) {
        this.id = id;
        return this;
    }

    public {{EntityName}}Fixture withName(String name) {
        this.name = name;
        return this;
    }

    public {{EntityName}}Fixture withDate(LocalDate date) {
        this.date = date;
        return this;
    }

    public {{EntityName}}Fixture withLocation(String location) {
        this.location = location;
        return this;
    }

    public {{EntityName}} buildEntity() {
        {{EntityName}} entity = new {{EntityName}}();
        entity.setId(id);
        entity.setName(name);
        entity.setDate(date);
        // Agregar más campos según la entidad
        return entity;
    }

    public {{EntityName}}RequestDto buildRequestDto() {
        {{EntityName}}RequestDto dto = new {{EntityName}}RequestDto();
        dto.setName(name);
        dto.setDate(date);
        // Agregar más campos según el DTO
        return dto;
    }

    public {{EntityName}}ResponseDto buildResponseDto() {
        {{EntityName}}ResponseDto dto = new {{EntityName}}ResponseDto();
        dto.setId(id);
        dto.setName(name);
        dto.setDate(date);
        // Agregar más campos según el DTO
        return dto;
    }

    // Métodos de conveniencia para casos comunes
    public static {{EntityName}} defaultEntity() {
        return builder().buildEntity();
    }

    public static {{EntityName}}RequestDto defaultRequestDto() {
        return builder().buildRequestDto();
    }

    public static {{EntityName}}ResponseDto defaultResponseDto() {
        return builder().buildResponseDto();
    }

    public static {{EntityName}} entityWithoutId() {
        return builder().withId(null).buildEntity();
    }

    public static {{EntityName}} futureEntity() {
        return builder()
                .withName("Future {{EntityName}}")
                .withDate(LocalDate.now().plusMonths(1))
                .buildEntity();
    }

    public static {{EntityName}} pastEntity() {
        return builder()
                .withName("Past {{EntityName}}")
                .withDate(LocalDate.now().minusMonths(1))
                .buildEntity();
    }
}
```

### Ejemplo de Uso de Fixtures

```java
@Test
void exampleUsingFixtures() {
    // Crear entidad por defecto
    {{EntityName}} default{{EntityName}} = {{EntityName}}Fixture.defaultEntity();
    
    // Crear entidad personalizada
    {{EntityName}} custom{{EntityName}} = {{EntityName}}Fixture.builder()
            .withId(2L)
            .withName("Custom {{EntityName}}")
            .withDate(LocalDate.of(2024, 6, 15))
            .buildEntity();
    
    // Crear DTO de request
    {{EntityName}}RequestDto requestDto = {{EntityName}}Fixture.builder()
            .withName("New {{EntityName}}")
            .buildRequestDto();
    
    // Usar casos predefinidos
    {{EntityName}} future = {{EntityName}}Fixture.futureEntity();
    {{EntityName}} past = {{EntityName}}Fixture.pastEntity();
}
```
