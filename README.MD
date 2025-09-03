

## 5) Arquitectura y decisiones técnicas

### 5.1 Diagrama general
El sistema se compone de dos capas principales:


- **Frontend (React + Vite):**
  - SPA que consume la API mediante `fetch` o `axios`.
  - Manejo de estado con hooks (`useState`, `useEffect`).
  - Componente principal: listado y creación de productos.
  - Configuración de CORS mediante variable `VITE_API_URL`.

- **Backend (FastAPI):**
  - Exposición de endpoints REST para **Productos** y **Categorías**.
  - Validaciones con **Pydantic** (unicidad, precios y cantidades > 0).
  - Manejo de errores con respuestas controladas (422, 404, 409).
  - Persistencia en base de datos relacional (SQLite para desarrollo, PostgreSQL para producción).

- **CORS:**
  - Habilitado en FastAPI para aceptar solicitudes desde `http://localhost:5173` (Vite).

---

### 5.2 Librerías clave

#### Backend
- **FastAPI** → framework para definir endpoints REST.
- **Pydantic** → validación de datos (reglas de negocio).
- **SQLAlchemy** → ORM para mapear entidades (Producto, Categoría).
- **Uvicorn** → servidor ASGI para correr la API.
- **pytest + httpx** → pruebas automáticas de endpoints.

#### Frontend
- **React + Vite** → desarrollo rápido de SPA.
- **axios** → consumo de la API (más cómodo que `fetch`).
- **React Router (opcional)** → navegación entre vistas.
- **TailwindCSS (opcional)** → estilos rápidos y consistentes.

---

### 5.3 Estrategia de estado en frontend
- **Estado local con hooks** (`useState`, `useEffect`).
- No se requiere Redux ni librerías externas dado el alcance del MVP.
- Ejemplo:
  - `productos` → lista obtenida desde el backend.
  - `loading` y `error` → control de estados al hacer fetch.

---

### 5.4 Manejo de errores
- **422 Unprocessable Entity** → datos inválidos (precio < 1, cantidad < 1).
- **404 Not Found** → recurso inexistente (producto o categoría no encontrada).
- **409 Conflict** → duplicados (nombre de producto o categoría ya existente).
- **200/201/204** → operaciones exitosas (lectura, creación, eliminación).

Frontend:
- Mostrar mensajes claros al usuario (ej. “El producto ya existe” o “Cantidad inválida”).
- Estados de **loading/error** visibles en pantalla.

---

### 5.5 Decisiones técnicas
- **Base de datos**: se usará SQLite en desarrollo por simplicidad, pero se puede migrar a PostgreSQL sin cambios mayores gracias a SQLAlchemy.
- **Estructura modular**: separar backend en carpetas (`models/`, `routes/`, `core/`) para mantener orden.
- **Validaciones en backend**: se prioriza la lógica de validación en Pydantic para evitar datos inconsistentes en la BD.
- **UI simple pero robusta**: empezar con listado + creación (vertical slice) y luego expandir a edición/eliminación.
- **Pruebas mínimas**: se incluirá al menos 1 prueba automatizada para verificar el CRUD.

---

### DIAGRAMA SPA
+------------------+                           +------------------------------+
|  Usuario         |  HTML/CSS/JS (SPA)        |  React (Vite) - Frontend     |
|  Navegador       +--------------------------> |  http://localhost:5173       |
+------------------+                           |  - Lista/crea productos      |
                                               |  - Estado: loading/error     |
                                               |  - fetch/axios               |
                                               +---------------+--------------+
                                                               |
                                                               | HTTP JSON (CORS)
                                                               v
                                               +---------------+--------------+
                                               |  FastAPI - Backend           |
                                               |  http://127.0.0.1:8000       |
                                               |  Endpoints:                  |
                                               |   - /productos (CRUD)        |
                                               |   - /categorias (CRUD)       |
                                               |  Validaciones Pydantic       |
                                               |  Códigos: 200/201/204/404/   |
                                               |           409/422            |
                                               +---------------+--------------+
                                                               |
                                                               | ORM (SQLAlchemy)
                                                               v
                                               +---------------+--------------+
                                               |   Base de datos              |
                                               |   SQLite (dev) / Postgres    |
                                               |   Tablas: Producto, Categoria|
                                               +------------------------------+

## SECUENCIA TIPICA
Usuario
  |
  | (1) Completa formulario "Nuevo producto"
  v
React SPA
  |
  | (2) POST /productos
  |     Body:
  |     { id_categoria, nombre_producto, precio, cantidad, fecha_creacion }
  v
FastAPI
  |
  | (3) Valida Pydantic:
  |     - nombre_producto único (409 si existe)
  |     - precio >= 1, cantidad >= 1 (422 si falla)
  | (4) INSERT via SQLAlchemy
  v
Base de datos
  |
  | (5) OK (id generado)
  v
FastAPI
  |
  | (6) 201 Created
  |     Location: /productos/{id}
  v
React SPA
  |
  | (7) GET /productos?offset=0&limit=10
  v
FastAPI -> DB -> FastAPI
  |
  | (8) 200 OK
  |     Header: X-Total-Count
  |     Body: [ ... productos ... ]
  v
React SPA (actualiza lista y muestra éxito)

## PUERTOS Y FLUJO DE RED
[ Navegador ]
     |
     |  Origin: http://localhost:5173
     v
[ React (Vite dev server) ]  --CORS-->  [ FastAPI (uvicorn) ]  -->  [ DB ]
                5173                         8000

## Mini-ERD
+-------------+        1        n +-------------+
|  Categoria  |------------------>|  Producto   |
| id (PK)     |                   | id (PK)     |
| nombre (*)  |                   | id_categoria|
| descripcion |                   | nombre (*)  |
+-------------+                   | precio      |
                                  | cantidad    |
                                  | fecha_crea  |
                                  +-------------+
(*) únicos

## 6) Plan del proyecto

### 6.1 Roadmap

**Semana 6 (Entregable 1 — MVP Alpha)**
- ✅ Definición del caso (inventario de productos).  
- ✅ Modelo de datos inicial (categorías y productos).  
- ✅ API diseñada (endpoints CRUD, validaciones).  
- ✅ Arquitectura y decisiones técnicas documentadas.  
- 🚧 Vertical slice en progreso (CRUD básico).  

**Semana 7–10**
- Implementar CRUD completo en backend con FastAPI + SQLAlchemy.  
- Añadir validaciones de negocio (unicidad, rangos, fechas).  
- Configurar CORS y variables de entorno (`VITE_API_URL`).  
- Frontend: listar y crear productos, mostrar loading/error.  
- Prueba automatizada básica con `pytest` + `httpx`.  

**Semana 11 (Entregable 2 — MVP completo)**  
- Backend y frontend listos para CRUD básico.  
- Paginación (`offset/limit`) en el GET de productos.  
- UI con búsqueda por nombre (debounce 300ms) y/o orden asc/desc.  
- Capturas de funcionamiento (UI + API).  

**Semana 12–15**
- Mejorar estilos de UI (TailwindCSS).  
- Manejo de más validaciones y reglas (ej. no borrar categorías con productos).  
- Tests adicionales (errores 404, 409, 422).  
- Documentación ampliada (ejemplos JSON de request/response).  
- Integración opcional con PostgreSQL.  

**Semana 16 (Entrega final)**  
- Demo funcional completa: CRUD productos + categorías.  
- Evidencia (capturas + video corto ≤3 min).  
- Repositorio final con estructura limpia y documentación clara.  
- Checklist final de entregable cumplido.  

---

## 6.2 Kanban En Imagen Subida al repositorio

