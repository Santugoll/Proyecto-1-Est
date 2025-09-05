## Punto 1
## Elevator pitch (3--5 lineas)

Una pequeña tienda necesita un sistema confiable para **gestionar su
inventario** sin errores de duplicados, precios mal ingresados ni
cantidades negativas. Nuestra app web permite **registrar, consultar,
editar y eliminar productos** con **validaciones claras** y alertas de
error (p.ej., producto duplicado). Esta pensada para el
**administrador** (gestiona el inventario) y el **empleado** (consulta
existencias) de forma **rapida y accesible** desde el navegador.

------------------------------------------------------------------------

## Usuarios/segmentos y principales casos de uso

-   **Administrador de la tienda**
    -   Registrar un producto con **nombre, categoria, precio,
        cantidad**.
    -   Editar **precio** o **cantidad** de productos existentes.
    -   Eliminar productos.
-   **Empleado**
    -   Buscar productos por **nombre** o **categoria** para verificar
        disponibilidad y stock.

**Reglas y criterios clave (resumen de historias de usuario):** -
**Duplicados**: si el nombre ya existe -> **HTTP 409**. - **Busqueda**:
siempre retorna **lista paginada** (puede ser vacia). - **Cantidad**:
nunca puede ser **negativa**.

------------------------------------------------------------------------

## Objetivos &  No-objetivos

**Objetivos** 1. CRUD de productos con validaciones de negocio
(duplicados, rangos validos, cantidades >= 0). 2. Busqueda/filtrado por
nombre y categoria con **paginacion**. 3. UI simple y responsiva,
enfocada en velocidad de captura y consulta. 4. API documentada
automaticamente (**Swagger UI** en FastAPI).

**No-objetivos (explicitamente fuera de alcance por ahora)** - Gestion
de usuarios/roles avanzada (mas alla de admin/empleado sin autenticacion
robusta). - Integracion con codigos de barras/lectores fisicos. -
Reportes avanzados (PDF/Excel), analitica o BI. - Integracion con
sistemas contables/ERP o facturacion. - Multi-sucursal o control de
movimientos/bitacora.

------------------------------------------------------------------------

## Metricas/KPIs de exito

-   **Exactitud de datos**: % de operaciones rechazadas por validacion
    (esperado: >= 95% de registros validos a la primera tras 1 semana de
    uso).
-   **Tiempo de operacion**: mediana de tiempo para **registrar** un
    producto (objetivo: \< 15 s) y para **buscar** uno (\< 3 s).
-   **Errores criticos**: 0 caidas en ambiente local durante sesiones de
    \> 1 hora.
-   **Cobertura funcional**: CRUD y busqueda con paginacion disponibles
    y probadas (checklist de QA >= 90% cumplida).
-   **Satisfaccion del usuario**: NPS interno de admin/empleado >= 8/10
    tras prueba piloto.

## Casos de uso (flujo minimo)

1.  **Registrar** -> Validar -> Crear -> Responder `201` con el producto.
2.  **Buscar** -> Filtros `name`, `category`, paginacion `page`, `size` ->
    Lista (puede ser vacia) y `total`.
3.  **Editar** -> Validar no negativo en cantidad -> Actualizar `price` o
    `quantity`.
4.  **Eliminar** -> Borrado por id/nombre.

## Convenciones de validacion (implementacion esperada)

-   **Nombre**: requerido, string no vacio, unico (case_insensitive
    sugerido).
-   **Categoria**: requerida, string no vacia.
-   **Precio**: numero \> 0 con hasta 2 decimales (rechazar valores no
    numericos).
-   **Cantidad**: entero >= 0 (rechazar negativos o no enteros).
-   **Codigos HTTP**:
    -   `201` creacion exitosa, `200` lectura/edicion/eliminacion
        exitosa.
    -   `400/422` validacion de esquema/valores.
    -   `409` conflicto por **nombre duplicado**.


## Must have (MVP inicial)

Registrar producto
Como administrador, quiero registrar un producto con nombre, precio y cantidad, para llevar control del inventario.

## Criterios de aceptación:

Si el nombre ya existe, el sistema devuelve error 409.

No se permite precio < 1 ni cantidad < 1.

El sistema responde con 201 y los datos del producto registrado.

Listar todos los productos
Como empleado, quiero listar todos los productos, para ver el inventario actual de la tienda.

Criterios de aceptación:

Devuelve lista completa de productos registrados.

Si no hay productos, retorna lista vacía.

El sistema responde con 200.

## Should have (valores añadidos al MVP)

Validaciones de precio y cantidad
Como administrador, quiero que el sistema valide el precio (mayor o igual a 1) y cantidad (entero ≥ 1), para evitar errores en el inventario.

Criterios de aceptación:

Valores inválidos → 422 Unprocessable Entity.

Precio y cantidad se guardan con formato correcto.

UI con alertas de error
Como usuario (admin/empleado), quiero recibir mensajes claros de error en la interfaz, para entender qué corregir.

Criterios de aceptación:

Mensajes visibles cuando el producto ya existe (409) o cuando hay valores inválidos (422).

Alertas visibles en pantalla sin romper el flujo de uso.

## Could have (no implementado todavía)

Editar producto (precio o cantidad).

Búsqueda por nombre o categoría.

Filtros avanzados de búsqueda (rango de precios, stock).

Exportar inventario a CSV.

## Won’t have (fuera de alcance en esta fase)

Gestión de usuarios y roles avanzados.
Se pospone para fases futuras.

## Mapa de versiones

MVP (Semana 11)

Historias 1 y 2 (Registrar y listar productos).

Historia 3 (Validaciones críticas).

Historia 4 (UI con alertas de error).

Versión final (Semana 16)

CRUD completo (editar, eliminar).

Búsqueda y filtros avanzados.

Exportar inventario a CSV.

Mejoras de UI/UX y pruebas extendidas.

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

### DIAGRAMA SPA SUBIDO AL REPOSITORIO

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

## 7 Estructura del proyecto
project/
├─ backend/
│  ├─ app/
│  │  ├─ api/routes/products.py
│  │  ├─ models/product.py
│  │  ├─ core/config.py
│  │  └─ main.py
│  ├─ requirements.txt
│  └─ tests/test_products.py
└─ frontend/
   ├─ src/
   │  ├─ api.js
   │  ├─ App.jsx
   │  └─ main.jsx
   ├─ .env.development
   ├─ package.json
   └─ vite.config.js

## requirements.txt
fastapi
uvicorn
sqlalchemy
pydantic
pytest
httpx






