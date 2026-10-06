# Contratos de la API — v1: Investigación

Base local de referencia:

```text
http://localhost:8000
```

La API de la versión 1 expone CRUD completo para las siguientes tablas:

- `area_conocimiento`
- `objetivo_desarrollo_sostenible`
- `area_aplicacion`
- `termino_clave`
- `universidad`
- `linea_investigacion`

---

## §0. El sobre, siempre el mismo

Las respuestas de la API utilizarán JSON.

### Éxito de lectura de listas

```json
{
    "tabla": "area_conocimiento",
    "limite": 1000,
    "total": 218,
    "datos": []
}
```

La misma estructura se utilizará para las demás tablas, cambiando el valor de
`tabla`, `total` y `datos`.

### Éxito de lectura individual

La consulta de un registro individual devuelve directamente el objeto:

```json
{
    "id": 1,
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}
```

> [!NOTE]
> El campo `activo` existe en la base de datos, pero no forma parte del
> contrato normal de los registros que recibe el frontend.

### Éxito de escritura

```json
{
    "estado": 200,
    "mensaje": "Operación realizada exitosamente.",
    "filasAfectadas": 1
}
```

### Error

```json
{
    "estado": 422,
    "mensaje": "Datos inválidos.",
    "errores": [
        "El campo nombre es obligatorio."
    ]
}
```

`errores[]` se utiliza específicamente para errores de validación de los
datos recibidos.

Para otros errores se utilizará `detalle`:

```json
{
    "estado": 404,
    "mensaje": "Registro no encontrado.",
    "detalle": "No existe el registro solicitado."
}
```

---

## §1. Traducción de errores a códigos HTTP

| Qué pasa | Quién lo detecta | Código |
|---|---|---|
| El cuerpo no tiene la forma esperada | Controller | `422` |
| Falta un campo obligatorio | Controller | `422` |
| Un campo tiene un tipo o formato inválido | Controller | `422` |
| La operación no tiene sentido para el recurso | Service | `400` |
| No se envió ningún campo para un PATCH | Service | `400` |
| El registro no existe o está inactivo | Service | `404` |
| La ruta existe pero no permite ese método | Router | `405` |
| Error inesperado de base de datos u otra excepción | Sistema | `500` |

La API no expondrá detalles internos innecesarios al usuario final.

---

## §2. `GET /` — diagnóstico

La ruta raíz permite comprobar que la API está funcionando.

```http
GET /
```

### Respuesta

```http
200 OK
```

```json
{
    "mensaje": "API de Investigación funcionando",
    "version": "v1",
    "modulo": "Investigación",
    "contratos": "/docs"
}
```

La ruta exacta de documentación podrá cambiar según la implementación final.

---

## §3. `GET /api/area_conocimiento` — listar

Obtiene las áreas de conocimiento activas.

```http
GET /api/area_conocimiento
```

### Parámetro opcional

`limite`

Debe ser un entero mayor que cero.

Si no se especifica, se utilizará:

```text
limite = 1000
```

### Respuesta con registros

```http
200 OK
```

```json
{
    "tabla": "area_conocimiento",
    "limite": 1000,
    "total": 218,
    "datos": [
        {
            "id": 1,
            "gran_area": "Ingeniería y Tecnología",
            "area": "Ingeniería de Sistemas",
            "disciplina": "Ingeniería de software"
        }
    ]
}
```

### Sin registros activos

```http
204 No Content
```

No se enviará cuerpo.

### Límite inválido

```http
400 Bad Request
```

```json
{
    "estado": 400,
    "mensaje": "Parámetros inválidos.",
    "detalle": "El límite debe ser un entero mayor que cero."
}
```

### Regla

Solamente se devolverán registros donde:

```text
activo = 1
```

Los registros eliminados lógicamente no aparecerán.

---

## §4. `GET /api/area_conocimiento/{id}` — obtener una

Obtiene un área de conocimiento activa mediante su identificador.

```http
GET /api/area_conocimiento/{id}
```

### Respuesta

```http
200 OK
```

```json
{
    "id": 1,
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}
```

### Registro inexistente o inactivo

```http
404 Not Found
```

```json
{
    "estado": 404,
    "mensaje": "Área de conocimiento no encontrada.",
    "detalle": "No existe el área de conocimiento con id = 1."
}
```

Un registro con `activo = 0` se considera inexistente para las consultas
normales de la API.

---

## §5. `POST /api/area_conocimiento` — crear

Crea un nuevo registro.

```http
POST /api/area_conocimiento
Content-Type: application/json
```

### Cuerpo

El cuerpo debe contener todos los campos editables necesarios.

```json
{
    "id": 219,
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}
```

`activo` no puede enviarse como campo de creación.

### Respuesta exitosa

```http
200 OK
```

```json
{
    "estado": 200,
    "mensaje": "Área de conocimiento creada exitosamente.",
    "filasAfectadas": 1
}
```

### Datos inválidos

```http
422 Unprocessable Entity
```

```json
{
    "estado": 422,
    "mensaje": "Datos inválidos.",
    "errores": [
        "El campo gran_area es obligatorio."
    ]
}
```

### Regla

El registro se crea con:

```text
activo = 1
```

El cliente no controla directamente este valor.

---

## §6. `PUT /api/area_conocimiento/{id}` — reemplazar

Reemplaza completamente los datos editables del registro.

```http
PUT /api/area_conocimiento/219
Content-Type: application/json
```

### Cuerpo

Todos los campos editables son obligatorios.

La llave primaria no se envía en el cuerpo porque se encuentra en la URL.

```json
{
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}
```

### Respuesta exitosa

```http
200 OK
```

```json
{
    "estado": 200,
    "mensaje": "Área de conocimiento reemplazada.",
    "filasAfectadas": 1
}
```

### Campo faltante

```http
422 Unprocessable Entity
```

```json
{
    "estado": 422,
    "mensaje": "Datos inválidos.",
    "errores": [
        "El campo gran_area es obligatorio."
    ]
}
```

### Registro inexistente

```http
404 Not Found
```

```json
{
    "estado": 404,
    "mensaje": "Área de conocimiento no encontrada.",
    "detalle": "No existe el área de conocimiento con id = 219."
}
```

### Regla

`PUT` significa reemplazo completo.

Por ejemplo, este cuerpo:

```json
{
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}
```

es inválido porque falta `gran_area`.

---

## §7. `PATCH /api/area_conocimiento/{id}` — actualizar parcialmente

Actualiza únicamente los campos enviados.

```http
PATCH /api/area_conocimiento/219
Content-Type: application/json
```

### Cuerpo

```json
{
    "disciplina": "Ingeniería de software"
}
```

También puede actualizar varios campos:

```json
{
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}
```

### Respuesta exitosa

```http
200 OK
```

```json
{
    "estado": 200,
    "mensaje": "Área de conocimiento actualizada.",
    "filasAfectadas": 1
}
```

### Cuerpo vacío

```http
400 Bad Request
```

```json
{
    "estado": 400,
    "mensaje": "Parámetros inválidos.",
    "detalle": "No se envió ningún campo para actualizar."
}
```

### Registro inexistente

```http
404 Not Found
```

```json
{
    "estado": 404,
    "mensaje": "Área de conocimiento no encontrada.",
    "detalle": "No existe el área de conocimiento con id = 219."
}
```

### Regla

La diferencia fundamental es:

```text
PUT   = todos los campos editables
PATCH = solamente los campos que cambian
```

El campo `activo` no puede actualizarse mediante `PATCH`.

---

## §8. `DELETE /api/area_conocimiento/{id}` — retirar

Realiza eliminación lógica.

```http
DELETE /api/area_conocimiento/219
```

No se eliminará físicamente la fila.

La operación será equivalente a:

```sql
UPDATE area_conocimiento
SET activo = 0
WHERE id = :id
  AND activo = 1;
```

### Respuesta exitosa

```http
200 OK
```

```json
{
    "estado": 200,
    "mensaje": "Área de conocimiento eliminada.",
    "filasAfectadas": 1
}
```

### Registro inexistente o ya retirado

```http
404 Not Found
```

```json
{
    "estado": 404,
    "mensaje": "Área de conocimiento no encontrada.",
    "detalle": "No existe un área de conocimiento activa con id = 219."
}
```

Por lo tanto, realizar dos veces el mismo `DELETE` no devuelve éxito en la
segunda operación.

---

## §9. Contrato de `objetivo_desarrollo_sostenible`

### Listar

```http
GET /api/objetivo_desarrollo_sostenible
```

**Respuesta**

```json
{
    "tabla": "objetivo_desarrollo_sostenible",
    "limite": 1000,
    "total": 17,
    "datos": [
        {
            "id": 1,
            "nombre": "Fin de la pobreza",
            "categoria": "Social"
        }
    ]
}
```

### Obtener

```http
GET /api/objetivo_desarrollo_sostenible/{id}
```

**Respuesta**

```json
{
    "id": 1,
    "nombre": "Fin de la pobreza",
    "categoria": "Social"
}
```

### Crear

```http
POST /api/objetivo_desarrollo_sostenible
```

**Cuerpo**

```json
{
    "id": 18,
    "nombre": "Nuevo objetivo",
    "categoria": "Social"
}
```

### Actualizar completamente

```http
PUT /api/objetivo_desarrollo_sostenible/{id}
```

**Cuerpo**

```json
{
    "nombre": "Nuevo objetivo",
    "categoria": "Social"
}
```

### Actualizar parcialmente

```http
PATCH /api/objetivo_desarrollo_sostenible/{id}
```

**Cuerpo**

```json
{
    "categoria": "Ambiental"
}
```

### Eliminar lógicamente

```http
DELETE /api/objetivo_desarrollo_sostenible/{id}
```

Todas las operaciones respetan:

```text
activo = 1
```

`activo` no forma parte del cuerpo enviado por el cliente.

---

## §10. Contrato de `area_aplicacion`

### Listar

```http
GET /api/area_aplicacion
```

**Respuesta**

```json
{
    "tabla": "area_aplicacion",
    "limite": 1000,
    "total": 21,
    "datos": [
        {
            "id": 1,
            "nombre": "Agricultura"
        }
    ]
}
```

### Obtener

```http
GET /api/area_aplicacion/{id}
```

**Respuesta**

```json
{
    "id": 1,
    "nombre": "Agricultura"
}
```

### Crear

```http
POST /api/area_aplicacion
```

**Cuerpo**

```json
{
    "id": 22,
    "nombre": "Nueva área"
}
```

### Actualizar completamente

```http
PUT /api/area_aplicacion/{id}
```

**Cuerpo**

```json
{
    "nombre": "Nueva área"
}
```

### Actualizar parcialmente

```http
PATCH /api/area_aplicacion/{id}
```

**Cuerpo**

```json
{
    "nombre": "Área actualizada"
}
```

### Eliminar lógicamente

```http
DELETE /api/area_aplicacion/{id}
```

La operación cambia:

```text
activo = 1
```

a:

```text
activo = 0
```

---

## §11. Contrato de `termino_clave`

Esta tabla utiliza una clave primaria de texto.

### Listar

```http
GET /api/termino_clave
```

**Respuesta**

```json
{
    "tabla": "termino_clave",
    "limite": 1000,
    "total": 10,
    "datos": [
        {
            "termino": "Inteligencia artificial",
            "termino_ingles": "Artificial intelligence"
        }
    ]
}
```

### Obtener

```http
GET /api/termino_clave/{termino}
```

El término deberá estar correctamente codificado en la URL.

Por ejemplo:

```text
/api/termino_clave/Inteligencia%20artificial
```

**Respuesta**

```json
{
    "termino": "Inteligencia artificial",
    "termino_ingles": "Artificial intelligence"
}
```

### Crear

```http
POST /api/termino_clave
```

**Cuerpo**

```json
{
    "termino": "Computación cuántica",
    "termino_ingles": "Quantum computing"
}
```

### Actualizar completamente

```http
PUT /api/termino_clave/{termino}
```

**Cuerpo**

```json
{
    "termino_ingles": "Quantum computing"
}
```

### Actualizar parcialmente

```http
PATCH /api/termino_clave/{termino}
```

**Cuerpo**

```json
{
    "termino_ingles": "Quantum computing"
}
```

### Eliminar lógicamente

```http
DELETE /api/termino_clave/{termino}
```

La eliminación no borra físicamente el término.

---

## §12. Contrato de `universidad`

### Listar

```http
GET /api/universidad
```

**Respuesta**

```json
{
    "tabla": "universidad",
    "limite": 1000,
    "total": 6,
    "datos": [
        {
            "id": 1,
            "nombre": "Universidad de ejemplo",
            "tipo": "Pública",
            "ciudad": "Medellín"
        }
    ]
}
```

### Obtener

```http
GET /api/universidad/{id}
```

**Respuesta**

```json
{
    "id": 1,
    "nombre": "Universidad de ejemplo",
    "tipo": "Pública",
    "ciudad": "Medellín"
}
```

### Crear

```http
POST /api/universidad
```

**Cuerpo**

```json
{
    "id": 7,
    "nombre": "Universidad de ejemplo",
    "tipo": "Pública",
    "ciudad": "Medellín"
}
```

### Actualizar completamente

```http
PUT /api/universidad/{id}
```

**Cuerpo**

```json
{
    "nombre": "Universidad actualizada",
    "tipo": "Pública",
    "ciudad": "Medellín"
}
```

### Actualizar parcialmente

```http
PATCH /api/universidad/{id}
```

**Cuerpo**

```json
{
    "ciudad": "Bogotá"
}
```

### Eliminar lógicamente

```http
DELETE /api/universidad/{id}
```

La API cambiará el estado lógico del registro a inactivo.

---

## §13. Contrato de `linea_investigacion`

### Listar

```http
GET /api/linea_investigacion
```

**Respuesta**

```json
{
    "tabla": "linea_investigacion",
    "limite": 1000,
    "total": 3,
    "datos": [
        {
            "id": 1,
            "nombre": "Inteligencia Artificial",
            "descripcion": "Investigación relacionada con inteligencia artificial."
        }
    ]
}
```

### Obtener

```http
GET /api/linea_investigacion/{id}
```

**Respuesta**

```json
{
    "id": 1,
    "nombre": "Inteligencia Artificial",
    "descripcion": "Investigación relacionada con inteligencia artificial."
}
```

### Crear

```http
POST /api/linea_investigacion
```

**Cuerpo**

No se envía `id` porque MariaDB lo genera automáticamente.

```json
{
    "nombre": "Inteligencia Artificial",
    "descripcion": "Investigación relacionada con inteligencia artificial."
}
```

**Respuesta**

```http
200 OK
```

```json
{
    "estado": 200,
    "mensaje": "Línea de investigación creada exitosamente.",
    "filasAfectadas": 1
}
```

### Actualizar completamente

```http
PUT /api/linea_investigacion/{id}
```

**Cuerpo**

```json
{
    "nombre": "Inteligencia Artificial",
    "descripcion": "Nueva descripción de la línea."
}
```

### Actualizar parcialmente

```http
PATCH /api/linea_investigacion/{id}
```

**Cuerpo**

```json
{
    "descripcion": "Descripción actualizada."
}
```

### Eliminar lógicamente

```http
DELETE /api/linea_investigacion/{id}
```

La operación cambia `activo` a `0`.

---

## §14. Resumen de endpoints de la v1

| Recurso | GET lista | GET uno | POST | PUT | PATCH | DELETE |
|---|---|---|---|---|---|---|
| `area_conocimiento` | `/api/area_conocimiento` | `/api/area_conocimiento/{id}` | Sí | Sí | Sí | Sí |
| `objetivo_desarrollo_sostenible` | `/api/objetivo_desarrollo_sostenible` | `/api/objetivo_desarrollo_sostenible/{id}` | Sí | Sí | Sí | Sí |
| `area_aplicacion` | `/api/area_aplicacion` | `/api/area_aplicacion/{id}` | Sí | Sí | Sí | Sí |
| `termino_clave` | `/api/termino_clave` | `/api/termino_clave/{termino}` | Sí | Sí | Sí | Sí |
| `universidad` | `/api/universidad` | `/api/universidad/{id}` | Sí | Sí | Sí | Sí |
| `linea_investigacion` | `/api/linea_investigacion` | `/api/linea_investigacion/{id}` | Sí | Sí | Sí | Sí |

---

## §15. Reglas comunes para todos los CRUD

Todos los recursos de la v1 deberán respetar las siguientes reglas:

1. Los listados solo muestran registros activos.
2. Las consultas individuales solo encuentran registros activos.
3. `POST` crea registros activos.
4. `PUT` reemplaza todos los campos editables.
5. `PATCH` modifica únicamente los campos enviados.
6. `DELETE` realiza eliminación lógica.
7. `activo` no puede ser modificado desde el cuerpo de las solicitudes.
8. Los campos obligatorios deben validarse antes de llegar al Repository.
9. Las consultas SQL utilizan parámetros preparados.
10. Los errores se devuelven en JSON.
11. Los Controllers son responsables del contrato HTTP.
12. Los Services contienen las reglas de negocio.
13. Los Repositories son responsables del acceso a MariaDB.

---

## §16. Cómo traduce el frontend estos desenlaces

| Respuesta de la API | Comportamiento del frontend |
|---|---|
| `200` en una lectura | Mostrar los registros o el registro solicitado |
| `204` | Mostrar mensaje indicando que todavía no existen registros activos |
| `200` en una creación | Mostrar mensaje de creación exitosa y actualizar el listado |
| `200` en una actualización | Mostrar mensaje de actualización exitosa |
| `200` en un `DELETE` | Mostrar mensaje de eliminación exitosa y actualizar el listado |
| `400` | Mostrar el mensaje de parámetros inválidos |
| `404` | Mostrar el mensaje de registro no encontrado |
| `422` | Mostrar cada mensaje contenido en `errores[]` |
| `405` | Mostrar que la operación HTTP no está permitida |
| `500` | Mostrar un mensaje genérico de error interno |
| La API no responde | Mostrar que el servicio no está disponible |

El frontend no debe mostrar directamente detalles técnicos de SQL, excepciones,
rutas internas o credenciales.

---

## §17. Separación entre API y presentación

Los siguientes elementos pertenecen exclusivamente al contrato de la API:

- rutas;
- métodos HTTP;
- códigos de estado;
- estructura JSON;
- nombres de campos;
- reglas de validación;
- mensajes de error.

La interfaz gráfica no deberá depender de la estructura interna de:

- Controllers;
- Services;
- Repositories;
- modelos internos;
- consultas SQL;
- conexión PDO.

La comunicación debe mantenerse mediante HTTP y JSON.

---

## §18. Criterio de aceptación del contrato v1

El contrato de la API de v1 se considera implementado cuando cada una de las
seis tablas permita realizar correctamente:

- [ ] `GET` lista
- [ ] `GET` uno
- [ ] `POST`
- [ ] `PUT`
- [ ] `PATCH`
- [ ] `DELETE` lógico

y cuando las siguientes situaciones hayan sido verificadas:

- [ ] registro existente;
- [ ] registro inexistente;
- [ ] registro inactivo;
- [ ] datos válidos;
- [ ] datos inválidos;
- [ ] cuerpo vacío;
- [ ] `PATCH` sin campos;
- [ ] `PUT` incompleto;
- [ ] eliminación lógica;
- [ ] segundo `DELETE` sobre el mismo registro;
- [ ] límite válido;
- [ ] límite inválido;
- [ ] error de método HTTP;
- [ ] error interno de la API.

El contrato deberá mantenerse estable durante la implementación de la v1.

Los cambios posteriores deberán documentarse antes de modificar el frontend o
los consumidores de la API.
