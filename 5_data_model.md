# Modelo de datos — v1 (MariaDB)

Base de datos: `investigacion`

En v1 son seis tablas **independientes** (sin relaciones entre ellas).

## Las tablas

### `area_conocimiento` (218 registros)

| Columna | Tipo | Notas |
|---|---|---|
| `id` | `INT` | Llave primaria |
| `gran_area` | `VARCHAR(60)` | Obligatorio |
| `area` | `VARCHAR(60)` | Obligatorio |
| `disciplina` | `VARCHAR(60)` | Obligatorio |
| `activo` | `TINYINT(1)` | Por defecto `1` (agregada) |

### `objetivo_desarrollo_sostenible` (17 registros)

| Columna | Tipo | Notas |
|---|---|---|
| `id` | `INT` | Llave primaria |
| `nombre` | `VARCHAR(60)` | Obligatorio |
| `categoria` | `VARCHAR(45)` | Obligatorio |
| `activo` | `TINYINT(1)` | Por defecto `1` (agregada) |

### `area_aplicacion` (21 registros)

| Columna | Tipo | Notas |
|---|---|---|
| `id` | `INT` | Llave primaria |
| `nombre` | `VARCHAR(60)` | Obligatorio |
| `activo` | `TINYINT(1)` | Por defecto `1` (agregada) |

### `termino_clave`

| Columna | Tipo | Notas |
|---|---|---|
| `termino` | `VARCHAR(30)` | Llave primaria (texto) |
| `termino_ingles` | `VARCHAR(30)` | Puede ser nulo |
| `activo` | `TINYINT(1)` | Por defecto `1` (agregada) |

No lleva `id`: el propio `termino` identifica el registro. Si tiene espacios, se codifican en la URL.

### `universidad` (6 registros)

| Columna | Tipo | Notas |
|---|---|---|
| `id` | `INT` | Llave primaria |
| `nombre` | `VARCHAR(60)` | Obligatorio |
| `tipo` | `VARCHAR(45)` | Obligatorio |
| `ciudad` | `VARCHAR(45)` | Obligatorio |
| `activo` | `TINYINT(1)` | Por defecto `1` (agregada) |

### `linea_investigacion`

| Columna | Tipo | Notas |
|---|---|---|
| `id` | `INT` | Llave primaria, `AUTO_INCREMENT` |
| `nombre` | `VARCHAR(45)` | Obligatorio |
| `descripcion` | `VARCHAR(256)` | Obligatorio |
| `activo` | `TINYINT(1)` | Por defecto `1` (agregada) |

El `id` lo genera MariaDB, así que el cliente **no lo envía** al crear.

## El campo `activo`

Es el **único cambio** que se hace al SQL oficial. Se agrega a las seis tablas:

```sql
`activo` TINYINT(1) NOT NULL DEFAULT 1
```

Debe quedar en `db/init.sql`.

- `1` = el registro está activo.
- `0` = el registro fue retirado.

Reglas:

- El cliente **no envía** `activo` en `POST`, `PUT` ni `PATCH`.
- `DELETE` lo cambia a `0` (el registro sigue en la base de datos).
- Los listados y consultas solo muestran `activo = 1`.

```sql
-- Retirar un registro
UPDATE area_conocimiento SET activo = 0 WHERE id = 10;

-- Listar
SELECT * FROM area_conocimiento WHERE activo = 1;
```

## Lo que NO se cambia

- No se renombran columnas ni se cambian tipos o llaves del SQL oficial.
- Si hay nombres raros en otras tablas (como `universidsad` o `nacionalidaad`), se dejan igual. No son de v1.

## Datos iniciales

Los datos de referencia se cargan desde `db/init.sql`, no desde el frontend, para poder probar los CRUD desde el principio.

## Las demás tablas

`docente`, `grupo_investigacion`, `semillero`, `participa_semillero`, `participa_grupo`, `semillero_linea`, `grupo_linea`, `ac_linea`, `ods_linea`, `aa_linea`, `rol`, `usuario` y `rol_usuario` **no se hacen en v1**:

- v2: las relaciones.
- v3: `usuario`, `rol` y `rol_usuario`.
- v4: consultas y dashboard.

## El modelo está listo cuando…

- [ ] Existen las seis tablas con sus columnas.
- [ ] Todas tienen `activo` con valor por defecto `1`.
- [ ] Los datos de referencia están cargados.
- [ ] Se puede crear, actualizar y retirar registros desde la API.
- [ ] Las consultas filtran `activo = 1`.
