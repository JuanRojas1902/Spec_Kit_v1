# Contrato de la API — v1

URL base local:

```text
http://localhost:8000
```

La API tiene CRUD para: `area_conocimiento`, `objetivo_desarrollo_sostenible`, `area_aplicacion`, `termino_clave`, `universidad` y `linea_investigacion`.

> [!NOTE]
> `activo` existe en la base de datos, pero **nunca** aparece en los JSON que entran o salen de la API.

## 1. Formato de las respuestas

Todo es JSON.

**Lista:**

```json
{
    "tabla": "area_conocimiento",
    "limite": 1000,
    "total": 218,
    "datos": []
}
```

**Un solo registro** (se devuelve el objeto directo):

```json
{
    "id": 1,
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}
```

**Escritura exitosa** (crear, cambiar o retirar):

```json
{
    "estado": 200,
    "mensaje": "Operación realizada exitosamente.",
    "filasAfectadas": 1
}
```

**Error de validación** (usa `errores`):

```json
{
    "estado": 422,
    "mensaje": "Datos inválidos.",
    "errores": ["El campo nombre es obligatorio."]
}
```

**Otros errores** (usan `detalle`):

```json
{
    "estado": 404,
    "mensaje": "Registro no encontrado.",
    "detalle": "No existe el registro solicitado."
}
```

## 2. Códigos de respuesta

| Situación | Código |
|---|---|
| Todo bien | `200` |
| Lista sin registros activos | `204` (sin cuerpo) |
| Parámetro inválido o `PATCH` vacío | `400` |
| Registro no existe o está retirado | `404` |
| Método no permitido en esa ruta | `405` |
| Cuerpo mal formado o campo faltante | `422` |
| Error inesperado | `500` |

## 3. Diagnóstico

```http
GET /
```

```json
{
    "mensaje": "API de Investigación funcionando",
    "version": "v1"
}
```

## 4. CRUD completo: `area_conocimiento`

### Listar

```http
GET /api/area_conocimiento
GET /api/area_conocimiento?limite=50
```

`limite` es opcional (por defecto 1000) y debe ser un número mayor que cero. Si no lo es, responde `400`.

### Ver uno

```http
GET /api/area_conocimiento/1
```

Si no existe o está retirado → `404`.

### Crear

```http
POST /api/area_conocimiento
```

```json
{
    "id": 219,
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}
```

Si falta un campo → `422`. El registro se crea con `activo = 1`.

### Reemplazar (`PUT`)

```http
PUT /api/area_conocimiento/219
```

```json
{
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}
```

Se envían **todos** los campos (menos el id, que va en la URL). Si falta alguno → `422`.

### Cambiar parte (`PATCH`)

```http
PATCH /api/area_conocimiento/219
```

```json
{
    "disciplina": "Ingeniería de software"
}
```

Solo se cambian los campos enviados. Si el cuerpo está vacío (`{}`) → `400`.

### Retirar (`DELETE`)

```http
DELETE /api/area_conocimiento/219
```

La fila no se borra, solo queda con `activo = 0`. Si se repite el mismo `DELETE`, la segunda vez responde `404`.

## 5. Las otras cinco tablas

Funcionan igual. Solo cambian los campos:

| Tabla | Campos | Ejemplo para crear (`POST`) |
|---|---|---|
| `objetivo_desarrollo_sostenible` | `id`, `nombre`, `categoria` | `{"id": 18, "nombre": "Nuevo objetivo", "categoria": "Social"}` |
| `area_aplicacion` | `id`, `nombre` | `{"id": 22, "nombre": "Nueva área"}` |
| `termino_clave` | `termino`, `termino_ingles` | `{"termino": "Computación cuántica", "termino_ingles": "Quantum computing"}` |
| `universidad` | `id`, `nombre`, `tipo`, `ciudad` | `{"id": 7, "nombre": "Universidad X", "tipo": "Pública", "ciudad": "Medellín"}` |
| `linea_investigacion` | `nombre`, `descripcion` (sin `id`) | `{"nombre": "IA", "descripcion": "Investigación en IA."}` |

Dos casos especiales:

- **`termino_clave`:** el término va en la URL y debe estar codificado: `/api/termino_clave/Inteligencia%20artificial`. En `PUT` solo se envía `termino_ingles`.
- **`linea_investigacion`:** el `id` lo genera MariaDB, no se envía al crear.

## 6. Resumen de rutas

| Recurso | Lista | Uno |
|---|---|---|
| `area_conocimiento` | `/api/area_conocimiento` | `/api/area_conocimiento/{id}` |
| `objetivo_desarrollo_sostenible` | `/api/objetivo_desarrollo_sostenible` | `/api/objetivo_desarrollo_sostenible/{id}` |
| `area_aplicacion` | `/api/area_aplicacion` | `/api/area_aplicacion/{id}` |
| `termino_clave` | `/api/termino_clave` | `/api/termino_clave/{termino}` |
| `universidad` | `/api/universidad` | `/api/universidad/{id}` |
| `linea_investigacion` | `/api/linea_investigacion` | `/api/linea_investigacion/{id}` |

Todas aceptan `GET`, `POST`, `PUT`, `PATCH` y `DELETE`.

## 7. Reglas para todas las tablas

1. Los listados y consultas solo muestran registros activos.
2. `POST` crea registros activos.
3. `PUT` pide todos los campos; `PATCH` solo los que cambian.
4. `DELETE` es borrado lógico.
5. `activo` no se puede enviar desde el cliente.
6. Los campos obligatorios se validan antes de llegar al repositorio.
7. El SQL usa consultas preparadas.

## 8. Qué hace el frontend con cada respuesta

| Respuesta | El frontend… |
|---|---|
| `200` al leer | Muestra los datos |
| `204` | Dice que todavía no hay registros |
| `200` al crear, cambiar o retirar | Muestra mensaje de éxito y actualiza la lista |
| `400` | Muestra el mensaje de parámetros inválidos |
| `404` | Dice que el registro no se encontró |
| `422` | Muestra cada mensaje de `errores` |
| `405` | Dice que la operación no está permitida |
| `500` | Muestra un mensaje de error general |
| La API no responde | Dice que el servicio no está disponible |

El frontend nunca muestra errores de SQL, rutas internas ni credenciales.

## 9. El contrato está cumplido cuando…

Las seis tablas permiten `GET` (lista y uno), `POST`, `PUT`, `PATCH` y `DELETE`, y se probaron estos casos:

- [ ] Registro que existe y registro que no existe.
- [ ] Registro retirado.
- [ ] Datos válidos y datos inválidos.
- [ ] `PATCH` vacío y `PUT` incompleto.
- [ ] Segundo `DELETE` sobre el mismo registro.
- [ ] Límite válido e inválido.
- [ ] Método no permitido.
