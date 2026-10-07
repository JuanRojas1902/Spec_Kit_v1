# Especificación — v1: Catálogos de Investigación

## 1. ¿Qué vamos a hacer?

En la v1 se construye el CRUD (crear, leer, actualizar y borrar) de **seis tablas de catálogo**:

1. `area_conocimiento`
2. `objetivo_desarrollo_sostenible`
3. `area_aplicacion`
4. `termino_clave`
5. `universidad`
6. `linea_investigacion`

Se usará:

- API en PHP (sin framework).
- Frontend en PHP con Bootstrap.
- Base de datos MariaDB.
- Programación orientada a objetos y arquitectura por capas.
- Borrado lógico (los registros no se borran de verdad).
- Contraseñas en variables de entorno (`.env`).

## 2. ¿Qué NO se hace en v1?

- Login, JWT, roles y usuarios.
- Consultas con varias tablas, dashboard y gráficas.
- Tablas con llaves foráneas.
- PWA y publicación en un servidor.

## 3. Las seis tablas

| Tabla | Campos | Llave primaria |
|---|---|---|
| `area_conocimiento` | `id`, `gran_area`, `area`, `disciplina` | `id` |
| `objetivo_desarrollo_sostenible` | `id`, `nombre`, `categoria` | `id` |
| `area_aplicacion` | `id`, `nombre` | `id` |
| `termino_clave` | `termino`, `termino_ingles` (opcional) | `termino` |
| `universidad` | `id`, `nombre`, `tipo`, `ciudad` | `id` |
| `linea_investigacion` | `id` (automático), `nombre`, `descripcion` | `id` |

Todos los campos son obligatorios, excepto `termino_ingles`.

## 4. Datos iniciales

| Tabla | Registros |
|---|---|
| `area_conocimiento` | 218 |
| `objetivo_desarrollo_sostenible` | 17 |
| `area_aplicacion` | 21 |
| `universidad` | 6 |
| `termino_clave` | Los de los datos de referencia |
| `linea_investigacion` | Los de los datos de referencia |

## 5. Lo que debe hacer la API

Para cada tabla:

| Acción | Método |
|---|---|
| Listar | `GET` |
| Ver uno | `GET` con el id |
| Crear | `POST` |
| Reemplazar todo | `PUT` |
| Cambiar solo algunos campos | `PATCH` |
| Retirar (borrado lógico) | `DELETE` |

Además, `GET /` sirve para comprobar que la API funciona.

Los detalles exactos están en [6_api_contract.md](6_api_contract.md).

## 6. Lo que debe hacer el frontend

Para cada tabla, una pantalla donde se pueda:

- Ver la lista de registros.
- Crear, editar y retirar registros.
- Ver mensajes de éxito y de error.

Debe usar Bootstrap y verse bien en pantallas pequeñas.

## 7. Validaciones

Se validan los campos obligatorios, el tamaño y el tipo de dato. El backend **siempre** valida, aunque el frontend ya lo haya hecho.

## 8. Arquitectura

```text
Frontend → API → Controlador → Servicio → Repositorio → MariaDB
```

- **Controlador:** recibe la petición HTTP y responde.
- **Servicio:** reglas de negocio.
- **Repositorio:** consultas SQL con PDO.

## 9. Seguridad

- Las contraseñas van en `.env`, que **no se sube a Git**.
- Se sube `.env.example` con los nombres de las variables, sin valores reales.

## 10. Git

- Dos repositorios privados: API y Frontend.
- El profesor `ccastro2050` debe estar invitado.
- Nadie trabaja directo en `main`: cada integrante tiene su rama y se une con Pull Request.

## 11. Versiones siguientes

| Versión | Qué trae |
|---|---|
| v1 | CRUD de 6 tablas |
| v2 | Las 10 tablas restantes con relaciones |
| v3 | Login, JWT y roles |
| v4 | Consultas, dashboard, PWA y publicación |

## 12. v1 está lista cuando…

- [ ] Los seis CRUD funcionan en la API y en el frontend.
- [ ] Los datos iniciales están cargados.
- [ ] El borrado es lógico y los retirados no aparecen en los listados.
- [ ] Hay un endpoint de diagnóstico (`GET /`).
- [ ] Se hicieron las pruebas principales.
- [ ] No hay contraseñas en el repositorio.
- [ ] Todo entró a `main` por Pull Request y se creó el tag `v1`.
