# Post-Contenido 1 — Refactorización con SOLID, DAO/DTO y @ControllerAdvice

**Programación Web — Unidad 11: Buenas Prácticas y Patrones de Diseño**  
Ingeniería de Sistemas — UDES 2026

---

## Descripción

Refactorización de una aplicación Spring Boot aplicando los principios SOLID (SRP y DIP), los patrones DAO y DTO para separar capas, un Factory para la construcción de objetos de respuesta, y manejo centralizado de excepciones mediante `@RestControllerAdvice` con respuestas de error estandarizadas.

---

## Requisitos previos

- Java 17+
- Maven 3.9.x
- Spring Boot 3.4.5
- IDE: IntelliJ IDEA o VS Code con Extension Pack for Java

---

## Estructura del Proyecto

```
apellido-post1-u11/
├── src/
│   └── main/
│       ├── java/com/empresa/catalogo/
│       │   ├── controller/
│       │   │   └── ProductoController.java
│       │   ├── service/
│       │   │   ├── ProductoService.java          (interfaz)
│       │   │   └── ProductoServiceImpl.java      (implementación)
│       │   ├── repository/
│       │   │   └── ProductoRepository.java
│       │   ├── dto/
│       │   │   ├── ProductoRequestDTO.java
│       │   │   └── ProductoResponseDTO.java
│       │   ├── entity/
│       │   │   └── Producto.java
│       │   ├── factory/
│       │   │   └── ProductoFactory.java
│       │   └── exception/
│       │       ├── ApiError.java
│       │       ├── GlobalExceptionHandler.java
│       │       └── RecursoNoEncontradoException.java
│       └── resources/
│           └── application.properties
└── pom.xml
```

---

## Arquitectura en Capas

```
Cliente HTTP
     │
     ▼
┌─────────────────────┐
│   ProductoController │  ← Recibe requests, delega al Service
└─────────┬───────────┘
          │ usa (interfaz)
          ▼
┌─────────────────────┐
│   ProductoService    │  ← Interfaz (DIP)
│   ProductoServiceImpl│  ← Lógica de negocio (SRP)
└─────────┬───────────┘
          │ usa
          ▼
┌─────────────────────┐   ┌─────────────────────┐
│ ProductoRepository   │   │   ProductoFactory    │
│ (DAO — JpaRepository)│   │ (conversión DTO↔Entity)│
└─────────┬───────────┘   └─────────────────────┘
          │
          ▼
┌─────────────────────┐
│   Producto (Entity)  │  ← Mapeada a tabla H2
└─────────────────────┘

Transversal:
┌──────────────────────────┐
│  GlobalExceptionHandler   │  ← @RestControllerAdvice
│  ApiError                 │  ← Respuesta de error estandarizada
└──────────────────────────┘
```

---

## Cómo ejecutar el proyecto

```bash
# Clonar el repositorio
git clone https://github.com/tu-usuario/apellido-post1-u11.git
cd apellido-post1-u11

# Compilar
mvn compile

# Ejecutar
mvn spring-boot:run
```

La aplicación queda disponible en `http://localhost:8081`.

---

## Endpoints disponibles

| Método | Endpoint | Descripción | Status exitoso |
|--------|----------|-------------|----------------|
| GET | `/api/productos` | Lista todos los productos activos | 200 |
| GET | `/api/productos/{id}` | Busca un producto por ID | 200 |
| POST | `/api/productos` | Crea un nuevo producto | 201 |
| DELETE | `/api/productos/{id}` | Elimina un producto | 204 |

### Ejemplo de request POST

```json
{
  "nombre": "Laptop",
  "precio": 3500000,
  "categoria": "ELECTRONICA"
}
```

### Ejemplo de response exitoso (201)

```json
{
  "id": 1,
  "nombre": "Laptop",
  "precio": 3500000.0,
  "categoria": "ELECTRONICA"
}
```

---

## Manejo de Errores

Todas las respuestas de error siguen el formato `ApiError`:

```json
{
  "status": 404,
  "error": "Not Found",
  "mensaje": "Producto con id 999 no encontrado.",
  "timestamp": "2026-05-12 16:00:00",
  "path": "/api/productos/999"
}
```

| Situación | Status | Ejemplo |
|-----------|--------|---------|
| Producto no existe | 404 | GET `/api/productos/999` |
| Campos inválidos | 400 | POST con body `{}` |
| Error inesperado | 500 | Error interno del servidor |

---

## Principios y Patrones aplicados

| Principio/Patrón | Dónde se aplica |
|------------------|-----------------|
| SRP (Single Responsibility) | Cada clase tiene una sola responsabilidad: Controller solo recibe requests, Service solo lógica de negocio, Repository solo acceso a datos |
| DIP (Dependency Inversion) | `ProductoController` depende de la interfaz `ProductoService`, no de la implementación |
| DAO | `ProductoRepository` extiende `JpaRepository` y encapsula el acceso a la base de datos |
| DTO | `ProductoRequestDTO` valida entrada; `ProductoResponseDTO` controla lo que se expone al cliente |
| Factory | `ProductoFactory` centraliza la conversión entre entidad y DTOs |
| @RestControllerAdvice | `GlobalExceptionHandler` captura excepciones globalmente y retorna `ApiError` estandarizado |

---

## Evidencias

| Evidencia | Descripción |
|-----------|-------------|
| `evidencia/compile-success.png` | `mvn compile` sin errores |
<img width="1920" height="1032" alt="idea64_ETbOErjhss" src="https://github.com/user-attachments/assets/9d010f5b-31a7-46f1-9d1b-90a6bd535e9f" />
| `evidencia/post-201.png` | POST exitoso retornando el DTO con `id` generado |
<img width="1920" height="1032" alt="Postman_sMeqwJiurO" src="https://github.com/user-attachments/assets/e7732b66-043c-43f5-9e69-dd8cc38a4042" />

| `evidencia/get-404.png` | GET `/api/productos/999` retorna status 404 con `ApiError` |
<img width="1920" height="1032" alt="Postman_NBwb4Em5ei" src="https://github.com/user-attachments/assets/99362072-a058-42dc-b4a7-c6e72a679770" />

| `evidencia/post-400.png` | POST con body vacío retorna status 400 con errores de validación |
<img width="1920" height="1032" alt="Postman_6PKYaLCow7" src="https://github.com/user-attachments/assets/2e59e7c4-0388-4be7-aca6-f1508747e28f" />



---

## Tecnologías usadas

| Tecnología | Versión | Uso |
|------------|---------|-----|
| Spring Boot | 3.4.5 | Framework base |
| Spring Web | — | Controladores REST |
| Spring Data JPA | — | Patrón DAO / acceso a datos |
| H2 Database | — | Base de datos en memoria |
| Spring Validation | — | Validación de DTOs con `@NotBlank`, `@Positive` |
