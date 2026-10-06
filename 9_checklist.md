# Checklist de cierre — v1: Investigación (PHP + MariaDB)

> [!IMPORTANT]
> Esto **no lo pasa un guion**. Las pruebas comprueban que el sistema
> responda; esta lista comprueba que esté bien hecho, que sea coherente
> con el Spec-Kit y que se entienda. La firma una persona.

---

## A. Funciona

- [ ] `docker compose up -d --build` levanta correctamente los servicios
      definidos para el proyecto.
- [ ] La API responde correctamente.
- [ ] El Frontend responde correctamente.
- [ ] Las seis tablas de v1 tienen CRUD completo:
      `area_conocimiento`,
      `objetivo_desarrollo_sostenible`,
      `area_aplicacion`,
      `termino_clave`,
      `universidad` y
      `linea_investigacion`.
- [ ] Los criterios de [2_spec.md](2_spec.md) pasan a mano utilizando
      los comandos y pasos de [7_quickstart.md](7_quickstart.md).
- [ ] Las pruebas de [8_tasks.md](8_tasks.md) fueron ejecutadas.
- [ ] `pruebas_humo/humo_front.py` termina correctamente.
- [ ] `pruebas/prueba_capas.php` termina correctamente.
- [ ] La prueba de capas se ejecutó al menos una vez con MariaDB apagada,
      demostrando que el servicio no depende directamente de la base de datos.
- [ ] El borrado de registros es lógico y no elimina físicamente los datos.
- [ ] Los registros con `activo = 0` no aparecen en los listados normales.
- [ ] Los datos de referencia fueron cargados correctamente.
- [ ] `area_conocimiento` contiene las 218 filas esperadas.
- [ ] `objetivo_desarrollo_sostenible` contiene las 17 filas esperadas.
- [ ] `area_aplicacion` contiene las 21 filas esperadas.
- [ ] `universidad` contiene las 6 filas esperadas.
- [ ] `termino_clave` contiene todos los datos de referencia entregados.
- [ ] `linea_investigacion` contiene todos los datos de referencia entregados.

---

## B. Las capas están de verdad cortadas

- [ ] Ningún archivo de `servicios/` menciona HTTP, códigos de estado,
      `$_GET`, `$_POST` o `$_SERVER`.
- [ ] Los servicios no construyen respuestas HTTP.
- [ ] Ningún archivo fuera de `repositorios/` ejecuta SQL directamente.
- [ ] Ningún archivo del Frontend utiliza PDO.
- [ ] Ningún archivo del Frontend contiene consultas SQL.
- [ ] Los repositorios son los responsables del acceso a MariaDB.
- [ ] Los repositorios utilizan PDO y prepared statements.
- [ ] `ensamblador.php` concentra la creación de las implementaciones
      concretas.
- [ ] Los controladores no contienen consultas SQL.
- [ ] Los modelos no contienen consultas SQL ni lógica HTTP.
- [ ] El Frontend no hace `require` de archivos internos de la API.
- [ ] El Frontend se comunica con la API mediante el cliente HTTP definido
      para el proyecto.
- [ ] El Frontend no contiene credenciales de MariaDB.
- [ ] El Frontend no necesita acceso directo a MariaDB.
- [ ] No existe `depends_on` innecesario del Frontend hacia MariaDB.
- [ ] La autenticación JWT no está mezclada con v1: corresponde a la versión
      v3.

---

## C. El contrato y la documentación dicen lo mismo que el código

- [ ] Cada ruta documentada en [6_contratos.md](6_contratos.md) existe
      realmente en la API.
- [ ] Cada ruta documenta tanto el caso exitoso como los errores posibles.
- [ ] Los métodos HTTP documentados coinciden con los métodos realmente
      implementados.
- [ ] Los códigos HTTP documentados coinciden con los códigos realmente
      devueltos.
- [ ] El formato de respuesta JSON coincide con el definido en
      [6_contratos.md](6_contratos.md).
- [ ] Los ejemplos del contrato son coherentes entre sí.
- [ ] La llave creada en un `POST` puede utilizarse posteriormente para
      consultar, modificar y retirar el mismo registro.
- [ ] Los comandos de [7_quickstart.md](7_quickstart.md) fueron ejecutados
      tal como están escritos.
- [ ] Los resultados obtenidos coinciden con los resultados indicados en
      `7_quickstart.md`.
- [ ] [5_data_model.md](5_data_model.md) coincide con la estructura real
      de las tablas.
- [ ] El campo `activo` está documentado como una modificación explícita
      respecto al SQL oficial del proyecto.
- [ ] [4_research.md](4_research.md) documenta las decisiones importantes
      tomadas durante v1.
- [ ] Cada decisión importante de `4_research.md` tiene su alternativa
      considerada o descartada.
- [ ] [8_tasks.md](8_tasks.md) refleja las tareas realmente realizadas.
- [ ] No existen contradicciones entre `2_spec.md`, `3_plan.md`,
      `4_research.md`, `5_data_model.md`, `6_contratos.md`,
      `7_quickstart.md` y `8_tasks.md`.

---

## D. La pantalla habla el idioma del usuario

- [ ] El usuario puede realizar el CRUD de las seis entidades desde el
      Frontend.
- [ ] Las pantallas no muestran detalles técnicos innecesarios.
- [ ] `PUT`, `PATCH`, `422`, `405`, `500` y otras decisiones internas de la
      API no aparecen como lenguaje principal de la interfaz.
- [ ] Las rutas `/api/...` no aparecen como información dirigida al usuario.
- [ ] Los mensajes de error están escritos de forma comprensible.
- [ ] Un error de validación no borra los datos que la persona había escrito.
- [ ] Un listado vacío no se presenta como un error del sistema.
- [ ] Cuando no existen registros, la pantalla informa que todavía no hay
      datos y ofrece la posibilidad de agregar uno.
- [ ] La interfaz diferencia correctamente entre crear y editar.
- [ ] La interfaz permite retirar un registro sin afirmar que fue eliminado
      físicamente.
- [ ] Se utiliza el término **«retirar»** cuando corresponde al borrado
      lógico.
- [ ] Los formularios muestran claramente los campos obligatorios.
- [ ] Los mensajes de éxito y error son visibles para la persona.
- [ ] La interfaz es usable en pantallas pequeñas.
- [ ] Bootstrap está disponible según la configuración definida en el
      proyecto y la interfaz no depende de un CDN para funcionar.

---

## E. Seguridad y configuración

- [ ] No existen contraseñas reales dentro del código fuente.
- [ ] No existen credenciales de MariaDB dentro del Frontend.
- [ ] `.env` aparece en `.gitignore`.
- [ ] `.env.example` está incluido en el repositorio.
- [ ] `.env.example` no contiene secretos reales.
- [ ] Las credenciales de MariaDB se obtienen mediante variables de entorno.
- [ ] No existen tokens JWT en el código: JWT corresponde a v3.
- [ ] No existe una clave `JWT_SECRET` real almacenada en el repositorio.
- [ ] Los mensajes de error no exponen contraseñas, tokens ni información
      sensible de configuración.
- [ ] No existen archivos temporales o archivos con secretos versionados.

---

## F. Base de datos y borrado lógico

- [ ] La base de datos se llama `investigacion`.
- [ ] Las seis tablas de v1 existen con los nombres definidos en el modelo.
- [ ] Las llaves primarias coinciden con el SQL oficial del proyecto.
- [ ] `termino_clave` mantiene su llave primaria de tipo texto.
- [ ] `linea_investigacion.id` mantiene su comportamiento `AUTO_INCREMENT`.
- [ ] Las seis tablas de v1 contienen el campo `activo` necesario para el
      borrado lógico.
- [ ] `activo` tiene valor activo por defecto.
- [ ] Los registros nuevos quedan activos.
- [ ] `DELETE` no ejecuta `DROP`, `DELETE FROM` ni elimina físicamente la fila.
- [ ] `DELETE` modifica el estado lógico del registro.
- [ ] Una fila retirada deja de aparecer en los listados normales.
- [ ] Una fila retirada no puede aparecer accidentalmente como activa.
- [ ] Las consultas normales aplican el filtro de registros activos.

---

## G. API y errores

- [ ] `GET` de una colección funciona.
- [ ] `GET` de un registro existente funciona.
- [ ] `GET` de un registro inexistente devuelve el error documentado.
- [ ] `POST` crea registros correctamente.
- [ ] `POST` con datos inválidos devuelve el error documentado.
- [ ] `PUT` reemplaza correctamente los datos definidos.
- [ ] `PATCH` modifica únicamente los campos enviados.
- [ ] `DELETE` realiza el retiro lógico.
- [ ] Un método HTTP no permitido devuelve `405`.
- [ ] Una ruta inexistente devuelve `404`.
- [ ] Un registro inexistente devuelve el `404` correspondiente.
- [ ] Los dos tipos de `404` pueden distinguirse correctamente.
- [ ] Los errores de validación devuelven el formato definido en
      `6_contratos.md`.
- [ ] Los errores internos no muestran información sensible.
- [ ] Las respuestas JSON mantienen una estructura uniforme.

---

## H. Revisión de las seis entidades

### `area_conocimiento`

- [ ] El CRUD funciona.
- [ ] Los 218 registros iniciales están cargados.
- [ ] `gran_area`, `area` y `disciplina` se muestran correctamente.
- [ ] El borrado lógico funciona.

### `objetivo_desarrollo_sostenible`

- [ ] El CRUD funciona.
- [ ] Los 17 ODS están cargados.
- [ ] `nombre` y `categoria` se muestran correctamente.
- [ ] El borrado lógico funciona.

### `area_aplicacion`

- [ ] El CRUD funciona.
- [ ] Los 21 registros están cargados.
- [ ] `nombre` se muestra correctamente.
- [ ] El borrado lógico funciona.

### `termino_clave`

- [ ] El CRUD funciona.
- [ ] La llave primaria textual funciona correctamente.
- [ ] `termino` y `termino_ingles` se muestran correctamente.
- [ ] El borrado lógico funciona.
- [ ] Los términos que contienen caracteres especiales se manejan
      correctamente.

### `universidad`

- [ ] El CRUD funciona.
- [ ] Las 6 universidades/registros de referencia están cargadas.
- [ ] `nombre`, `tipo` y `ciudad` se muestran correctamente.
- [ ] El borrado lógico funciona.

### `linea_investigacion`

- [ ] El CRUD funciona.
- [ ] Los datos de referencia están cargados.
- [ ] El `id` se genera correctamente mediante `AUTO_INCREMENT`.
- [ ] `nombre` y `descripcion` se muestran correctamente.
- [ ] El borrado lógico funciona.

---

## I. Revisión final de Git

- [ ] Los cambios de v1 fueron realizados en ramas de trabajo y no
      directamente sobre `main`.
- [ ] Los cambios llegaron a `main` mediante Pull Request.
- [ ] El repositorio no contiene secretos.
- [ ] El repositorio contiene todos los documentos del Spec-Kit.
- [ ] Los archivos tienen nombres consistentes.
- [ ] Los enlaces internos de los documentos funcionan.
- [ ] El Pull Request de cierre fue revisado.
- [ ] El commit final de v1 está identificado.
- [ ] Se creó el tag:

```text
v1
```

---

## J. Antes de pasar a v2

- [ ] Las seis entidades de v1 están completamente funcionales.
- [ ] No quedan errores conocidos que impidan utilizar el CRUD.
- [ ] No quedan tareas críticas pendientes de v1.
- [ ] La documentación corresponde al código real.
- [ ] La base de datos corresponde al modelo documentado.
- [ ] La API corresponde al contrato documentado.
- [ ] El Frontend corresponde al contrato de la API.
- [ ] Una persona diferente al desarrollador pudo levantar el proyecto
      siguiendo `7_quickstart.md`.
- [ ] Esa persona pudo realizar un CRUD completo sin modificar el código.
- [ ] El equipo puede explicar las decisiones principales tomadas durante
      v1.
- [ ] El equipo puede explicar qué quedó deliberadamente para v2, v3 y v4.

---

## Firma

**Revisó:** ______________________________________

**Fecha:** _______________________________________

**Resultado de la revisión:**

- [ ] APROBADO PARA CERRAR v1
- [ ] APROBADO CON OBSERVACIONES
- [ ] NO APROBADO

**Observaciones:**

&nbsp;

&nbsp;

**Lo que quedó anotado para la v2:**

&nbsp;

&nbsp;

**Firma:** _______________________________________
