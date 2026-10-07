# Tareas — v1

Se trabaja **de adentro hacia afuera**. Cada fase termina con algo que se puede comprobar.

Las seis tablas: `area_conocimiento`, `objetivo_desarrollo_sostenible`, `area_aplicacion`, `termino_clave`, `universidad`, `linea_investigacion`.

## Fase 0 — Docker y base de datos

- [ ] `docker-compose.yml` con MariaDB, API y frontend.
- [ ] `db/init.sql` crea la base `investigacion` y las seis tablas.
- [ ] Las seis tablas tienen el campo `activo`.
- [ ] Los datos de referencia quedan cargados.
- [ ] Las credenciales están en `.env`, que está en `.gitignore`.
- [ ] `.env.example` tiene solo los nombres de las variables.

**Verificación:**

```powershell
docker compose up -d
docker compose ps
```

```sql
USE investigacion;
SHOW TABLES;
SELECT COUNT(*) FROM area_conocimiento;  -- 218
```

## Fase 1 — Modelos

- [ ] Un modelo por tabla en `modelos/` (por ejemplo `AreaConocimiento.php`).
- [ ] Propiedades privadas, con getters, setters y `toArray()`.
- [ ] Sin SQL y sin HTTP.

## Fase 2 — Interfaces

- [ ] `IRepositorio...` para cada tabla, en `repositorios/`.
- [ ] `IServicio...` para cada tabla, en `servicios/`.
- [ ] Definen: listar, consultar uno, crear, reemplazar (`PUT`), cambiar parte (`PATCH`) y retirar.

## Fase 3 — Repositorios

- [ ] Un `Repositorio...MariaDB.php` por tabla.
- [ ] Usan PDO y `prepare()` (nunca concatenar valores).
- [ ] Filtran `activo = 1` en listados y consultas por id.
- [ ] El borrado se hace con `UPDATE`, nunca con `DELETE FROM`.

**Verificación:** desde el contenedor de la API, listar `area_conocimiento` debe dar 218 registros.

## Fase 4 — Servicios

- [ ] Un `Servicio...php` por tabla.
- [ ] Reciben el repositorio por el constructor.
- [ ] Validan los datos obligatorios.
- [ ] Lanzan una excepción si el registro no se encuentra.
- [ ] No usan nada de HTTP (`$_POST`, `header()`, etc.).
- [ ] `servicios/ensamblador.php` crea los objetos reales.

## Fase 5 — Controladores

- [ ] Un `Controlador...php` por tabla.
- [ ] Leen el JSON y validan que tenga la forma correcta.
- [ ] Distinguen `POST`/`PUT` (todos los campos) de `PATCH` (solo los enviados).
- [ ] Convierten las excepciones en códigos HTTP.
- [ ] Responden con el formato de [6_api_contract.md](6_api_contract.md).
- [ ] No tienen SQL.

## Fase 6 — Rutas (`index.php`)

- [ ] Rutas `GET`, `POST`, `PUT`, `PATCH` y `DELETE` para las seis tablas.
- [ ] `405` si el método no está permitido.
- [ ] `404` distinto para ruta inexistente y registro no encontrado.
- [ ] Todas las respuestas en JSON.

**Verificación:** seguir [7_quickstart.md](7_quickstart.md); el CRUD debe funcionar para las seis tablas.

## Fase 7 — Prueba de capas

- [ ] `pruebas/prueba_capas.php`.
- [ ] Un repositorio falso en memoria que implementa la interfaz.
- [ ] Prueba: crear, consultar, `PUT`, `PATCH`, datos inválidos, registro inexistente y borrado lógico.

```powershell
docker compose exec api-investigacion php pruebas/prueba_capas.php
```

## Fase 8 — Frontend

- [ ] `front_php/cliente_api.php`: único archivo que habla con la API.
- [ ] `front_php/index.php`: rutas de las pantallas.
- [ ] Vistas: plantilla general, inicio, lista, formulario y página 404.
- [ ] Pantallas de las seis tablas (listar, crear, editar y retirar).
- [ ] Muestra mensajes de éxito y de error.
- [ ] Bootstrap guardado localmente (sin depender de internet).
- [ ] El frontend no usa PDO ni SQL.

## Fase 9 — Pruebas juntas

Para cada tabla:

- [ ] Crear desde el frontend y ver que aparece en la API.
- [ ] Editar y comprobar el cambio.
- [ ] Retirar y comprobar que no aparece en la lista.
- [ ] Comprobar que la fila sigue en MariaDB con `activo = 0`.

Errores:

- [ ] No existe → `404`.
- [ ] Cuerpo inválido → `422`.
- [ ] Método no permitido → `405`.

## Fase 10 — Datos de referencia

- [ ] `area_conocimiento`: 218.
- [ ] `objetivo_desarrollo_sostenible`: 17.
- [ ] `area_aplicacion`: 21.
- [ ] `universidad`: 6.
- [ ] `termino_clave` y `linea_investigacion`: todos los de los datos entregados.
- [ ] Sin duplicados y visibles en el frontend.

## Fase 11 — Seguridad

- [ ] No hay contraseñas en el código ni en el frontend.
- [ ] `.env` no está en Git, `.env.example` sí.
- [ ] No hay JWT todavía (va en v3).
- [ ] Los errores no muestran datos sensibles.

## Fase 12 — Colección de pruebas (opcional)

- [ ] Una colección en Postman (o similar) con listar, ver uno, crear, `PUT`, `PATCH`, `DELETE`, no existe, datos inválidos y método no permitido.

## Fase 13 — Cerrar v1

- [ ] Las seis tablas funcionan en API y frontend.
- [ ] El borrado es lógico.
- [ ] Los datos de referencia están cargados.
- [ ] La prueba de capas y las pruebas manuales pasan.
- [ ] Los documentos 2 a 7 coinciden con lo que se hizo.
- [ ] Este archivo está actualizado.
- [ ] Se revisó y firmó [9_checklist.md](9_checklist.md).

**Criterio final:** otra persona puede seguir `7_quickstart.md`, levantar el proyecto y hacer el CRUD de las seis tablas sin tocar el código.

Después:

- [ ] Commit final.
- [ ] Pull Request revisado y unido a `main`.
- [ ] Crear el tag `v1`.
