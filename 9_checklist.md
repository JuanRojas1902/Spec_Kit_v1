# Checklist de cierre — v1

> [!IMPORTANT]
> Esta lista **no la pasa un programa**: la revisa una persona. Las pruebas dicen si el sistema responde; esta lista dice si está bien hecho y se entiende.

## A. Funciona

- [ ] `docker compose up -d --build` levanta todo.
- [ ] La API y el frontend responden.
- [ ] Las seis tablas tienen CRUD completo.
- [ ] Se siguieron los pasos de [7_quickstart.md](7_quickstart.md) y dieron lo esperado.
- [ ] `pruebas/prueba_capas.php` pasa.
- [ ] `pruebas_humo/humo_front.py` pasa.
- [ ] El borrado es lógico y los registros con `activo = 0` no salen en los listados.
- [ ] Datos cargados: 218, 17, 21 y 6, más `termino_clave` y `linea_investigacion`.

## B. Las capas están separadas

- [ ] Los servicios no usan HTTP (`$_GET`, `$_POST`, `$_SERVER`, `header()`).
- [ ] Solo los repositorios ejecutan SQL.
- [ ] Los controladores y modelos no tienen SQL.
- [ ] El frontend no usa PDO ni SQL, y solo habla con la API.
- [ ] `ensamblador.php` es quien crea los objetos reales.
- [ ] No hay JWT mezclado en v1.

## C. La documentación dice lo mismo que el código

- [ ] Cada ruta de [6_api_contract.md](6_api_contract.md) existe en la API.
- [ ] Los códigos y el formato JSON del contrato coinciden con lo que devuelve la API.
- [ ] [5_data_model.md](5_data_model.md) coincide con las tablas reales.
- [ ] `activo` está documentado como cambio respecto al SQL oficial.
- [ ] [4_research.md](4_research.md) y [8_tasks.md](8_tasks.md) reflejan lo que de verdad se hizo.
- [ ] No hay contradicciones entre los documentos 2 a 8.

## D. La pantalla es clara para el usuario

- [ ] Se puede hacer el CRUD de las seis tablas desde el frontend.
- [ ] No se muestran términos técnicos (`PUT`, `PATCH`, `422`, rutas `/api/...`).
- [ ] Los mensajes de error se entienden.
- [ ] Si hay un error de validación, no se pierde lo que se escribió.
- [ ] Una lista vacía dice "todavía no hay datos" y deja agregar uno.
- [ ] Se usa la palabra **«retirar»** para el borrado lógico.
- [ ] Los campos obligatorios se ven claramente.
- [ ] Se ve bien en pantallas pequeñas.
- [ ] Bootstrap funciona sin depender de internet.

## E. Seguridad

- [ ] No hay contraseñas ni tokens en el código.
- [ ] El frontend no tiene credenciales de la base de datos.
- [ ] `.env` está en `.gitignore` y `.env.example` está en el repositorio.
- [ ] Los errores no muestran información sensible.

## F. Base de datos

- [ ] La base se llama `investigacion`.
- [ ] Las seis tablas existen y las llaves primarias son las del SQL oficial.
- [ ] `termino_clave` mantiene su llave de texto.
- [ ] `linea_investigacion.id` es `AUTO_INCREMENT`.
- [ ] Las seis tablas tienen `activo` con valor por defecto `1`.
- [ ] `DELETE` solo cambia `activo` a `0`.

## G. API y errores

- [ ] Funcionan `GET` (lista y uno), `POST`, `PUT`, `PATCH` y `DELETE`.
- [ ] Datos inválidos → `422`.
- [ ] Registro que no existe → `404`.
- [ ] Método no permitido → `405`.
- [ ] Ruta que no existe → `404` (distinguible del anterior).
- [ ] Las respuestas siempre son JSON con el mismo formato.

## H. Las seis tablas

- [ ] `area_conocimiento`: CRUD, 218 registros y borrado lógico.
- [ ] `objetivo_desarrollo_sostenible`: CRUD, 17 registros y borrado lógico.
- [ ] `area_aplicacion`: CRUD, 21 registros y borrado lógico.
- [ ] `termino_clave`: CRUD, llave de texto y caracteres especiales funcionan.
- [ ] `universidad`: CRUD, 6 registros y borrado lógico.
- [ ] `linea_investigacion`: CRUD y `id` automático.

## I. Git

- [ ] Se trabajó en ramas y no directo en `main`.
- [ ] Todo llegó a `main` por Pull Request.
- [ ] No hay secretos en el repositorio.
- [ ] Están todos los documentos (2 a 9) y sus enlaces funcionan.
- [ ] Se creó el tag `v1`.

## J. Antes de pasar a v2

- [ ] Las seis tablas funcionan completamente y no quedan errores graves.
- [ ] La documentación, la base de datos y la API corresponden al código real.
- [ ] Otra persona levantó el proyecto con `7_quickstart.md` y pudo hacer un CRUD sin tocar el código.
- [ ] El equipo sabe explicar las decisiones de v1 y qué queda para v2, v3 y v4.

---

## Firma

**Revisó:** ______________________________________

**Fecha:** _______________________________________

**Resultado:**

- [ ] APROBADO PARA CERRAR v1
- [ ] APROBADO CON OBSERVACIONES
- [ ] NO APROBADO

**Observaciones:**

&nbsp;

&nbsp;

**Anotado para la v2:**

&nbsp;

&nbsp;

**Firma:** _______________________________________
