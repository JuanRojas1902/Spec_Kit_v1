# Tareas — v1: Investigación (PHP + MariaDB)

El orden importa: **de adentro hacia afuera**. Cada fase termina con algo que
se puede comprobar, no con «ya quedó».

La versión v1 se cierra cuando las seis tablas iniciales tienen CRUD completo
API + Frontend, funcionan con borrado lógico, cargan los datos de referencia
del proyecto y pasan las verificaciones del `7_quickstart.md`.

Las seis tablas de v1 son:

1. `area_conocimiento`
2. `objetivo_desarrollo_sostenible`
3. `area_aplicacion`
4. `termino_clave`
5. `universidad`
6. `linea_investigacion`

---

## Fase 0 — El compose y la base

- [ ] `docker-compose.yml` con los servicios necesarios para MariaDB, API,
      Frontend y administración de base de datos.
- [ ] `db/init.sql` derivado del script oficial del proyecto.
- [ ] El script crea la base de datos `investigacion`.
- [ ] El script crea las seis tablas de v1.
- [ ] Las seis tablas de v1 tienen el campo `activo` para soportar el
      borrado lógico, documentado como cambio de esquema en `5_data_model.md`
      y `4_research.md`.
- [ ] Los datos de referencia del proyecto quedan cargados en la base.
- [ ] Las credenciales de la base de datos no aparecen escritas directamente
      en el código.
- [ ] Las variables sensibles se manejan mediante variables de entorno.
- [ ] `.env` está incluido en `.gitignore`.
- [ ] `.env.example` contiene únicamente los nombres de las variables
      necesarias, sin contraseñas reales.

**Verificación:**

```powershell
docker compose up -d
docker compose ps
```

Después comprobar que la base existe:

```sql
USE investigacion;

SHOW TABLES;
```

Deben aparecer, como mínimo, las seis tablas de v1:

```text
area_conocimiento
objetivo_desarrollo_sostenible
area_aplicacion
termino_clave
universidad
linea_investigacion
```

Comprobar las cantidades conocidas:

```sql
SELECT COUNT(*) AS total FROM area_conocimiento;
-- 218

SELECT COUNT(*) AS total FROM objetivo_desarrollo_sostenible;
-- 17

SELECT COUNT(*) AS total FROM area_aplicacion;
-- 21

SELECT COUNT(*) AS total FROM universidad;
-- 6
```

Los totales de `termino_clave` y `linea_investigacion` deben coincidir con
los datos de referencia entregados para el proyecto.

---

## Fase 1 — Los modelos

Crear los modelos correspondientes a las seis tablas de v1.

- [ ] `modelos/AreaConocimiento.php`.
- [ ] `modelos/ObjetivoDesarrolloSostenible.php`.
- [ ] `modelos/AreaAplicacion.php`.
- [ ] `modelos/TerminoClave.php`.
- [ ] `modelos/Universidad.php`.
- [ ] `modelos/LineaInvestigacion.php`.

Cada modelo debe:

- [ ] Tener propiedades privadas.
- [ ] Tener getters y setters para los campos editables.
- [ ] Mantener la llave primaria según el modelo de datos.
- [ ] Tener `toArray()`.
- [ ] Representar únicamente los datos propios de la entidad.
- [ ] No contener lógica HTTP.
- [ ] No contener consultas SQL.
- [ ] No manejar directamente la conexión a MariaDB.

> [!NOTE]
> El campo `activo` se utiliza para el borrado lógico de la persistencia, pero
> no se considera parte de la ficha funcional que el usuario edita.

---

## Fase 2 — Las interfaces

Crear los contratos que separan la lógica de negocio del acceso a datos.

### Repositorios

- [ ] `repositorios/IRepositorioAreaConocimiento.php`.
- [ ] `repositorios/IRepositorioObjetivoDesarrolloSostenible.php`.
- [ ] `repositorios/IRepositorioAreaAplicacion.php`.
- [ ] `repositorios/IRepositorioTerminoClave.php`.
- [ ] `repositorios/IRepositorioUniversidad.php`.
- [ ] `repositorios/IRepositorioLineaInvestigacion.php`.

### Servicios

- [ ] `servicios/IServicioAreaConocimiento.php`.
- [ ] `servicios/IServicioObjetivoDesarrolloSostenible.php`.
- [ ] `servicios/IServicioAreaAplicacion.php`.
- [ ] `servicios/IServicioTerminoClave.php`.
- [ ] `servicios/IServicioUniversidad.php`.
- [ ] `servicios/IServicioLineaInvestigacion.php`.

Los contratos deben definir las operaciones necesarias para:

- [ ] listar.
- [ ] consultar un registro.
- [ ] crear.
- [ ] reemplazar mediante `PUT`.
- [ ] actualizar parcialmente mediante `PATCH`.
- [ ] eliminar lógicamente.

**Verificación:**

La prueba de capas de la Fase 8 debe poder utilizar los contratos sin
depender de las implementaciones concretas.

---

## Fase 3 — Los repositorios

Crear los repositorios MariaDB para las seis tablas.

- [ ] `repositorios/RepositorioAreaConocimientoMariaDB.php`.
- [ ] `repositorios/RepositorioObjetivoDesarrolloSostenibleMariaDB.php`.
- [ ] `repositorios/RepositorioAreaAplicacionMariaDB.php`.
- [ ] `repositorios/RepositorioTerminoClaveMariaDB.php`.
- [ ] `repositorios/RepositorioUniversidadMariaDB.php`.
- [ ] `repositorios/RepositorioLineaInvestigacionMariaDB.php`.

Cada repositorio debe:

- [ ] Utilizar PDO.
- [ ] Utilizar prepared statements.
- [ ] No construir consultas concatenando datos recibidos del usuario.
- [ ] Filtrar los registros activos en las consultas normales.
- [ ] Aplicar `activo = 1` en listados y consultas por identificador.
- [ ] Realizar el borrado lógico mediante `UPDATE`.
- [ ] No eliminar físicamente los registros.
- [ ] Mantener la responsabilidad SQL dentro del repositorio.
- [ ] Respetar las llaves primarias de cada tabla.
- [ ] Respetar los tipos de datos definidos en `5_data_model.md`.

Para consultas con `LIMIT`:

- [ ] Utilizar `bindValue(..., PDO::PARAM_INT)` cuando corresponda.

Para detectar correctamente filas afectadas:

- [ ] Configurar `PDO::MYSQL_ATTR_FOUND_ROWS => true` cuando corresponda
      al diseño del repositorio.

**Verificación:**

Desde el contenedor de la API se debe poder obtener el total de registros
activos de cada tabla.

Ejemplo:

```powershell
docker compose exec <servicio-api> php -r "require 'servicios/ensamblador.php'; print_r(count(crearServicioAreaConocimiento()->listar(1000)));"
```

El resultado esperado para `area_conocimiento` es:

```text
218
```

Repetir la comprobación para las otras cinco entidades.

---

## Fase 4 — Los servicios

Crear los servicios correspondientes a las seis entidades.

- [ ] `servicios/ServicioAreaConocimiento.php`.
- [ ] `servicios/ServicioObjetivoDesarrolloSostenible.php`.
- [ ] `servicios/ServicioAreaAplicacion.php`.
- [ ] `servicios/ServicioTerminoClave.php`.
- [ ] `servicios/ServicioUniversidad.php`.
- [ ] `servicios/ServicioLineaInvestigacion.php`.

Cada servicio debe:

- [ ] Recibir un repositorio mediante inyección de dependencias.
- [ ] Validar las reglas de negocio.
- [ ] Validar datos obligatorios.
- [ ] Validar datos que no puedan estar vacíos.
- [ ] Lanzar excepciones de negocio cuando corresponda.
- [ ] Utilizar una excepción específica para registros no encontrados.
- [ ] No conocer códigos HTTP.
- [ ] No construir respuestas HTTP.
- [ ] No leer directamente `$_SERVER`, `$_POST` u otros datos HTTP.

Crear o completar:

- [ ] `servicios/ensamblador.php`.

El ensamblador debe centralizar la creación de las implementaciones concretas
y evitar que el controlador tenga que construir directamente repositorios y
servicios.

> [!IMPORTANT]
> El servicio conoce las reglas del sistema, pero no sabe si fue llamado
> desde HTTP, una prueba automática o cualquier otra interfaz.

---

## Fase 5 — Los controladores

Crear los controladores de las seis entidades.

- [ ] `controladores/ControladorAreaConocimiento.php`.
- [ ] `controladores/ControladorObjetivoDesarrolloSostenible.php`.
- [ ] `controladores/ControladorAreaAplicacion.php`.
- [ ] `controladores/ControladorTerminoClave.php`.
- [ ] `controladores/ControladorUniversidad.php`.
- [ ] `controladores/ControladorLineaInvestigacion.php`.

Cada controlador debe encargarse de:

- [ ] Leer los datos de la petición.
- [ ] Decodificar el JSON cuando corresponda.
- [ ] Validar que el cuerpo tenga un formato correcto.
- [ ] Validar las columnas permitidas.
- [ ] Llamar al servicio correspondiente.
- [ ] Traducir excepciones de negocio a respuestas HTTP.
- [ ] Construir el envelope de respuesta definido en `6_contratos.md`.
- [ ] No ejecutar SQL directamente.

Implementar un mecanismo común para validar los campos de:

- [ ] `POST`.
- [ ] `PUT`.
- [ ] `PATCH`.

La validación debe distinguir entre:

- [ ] Campos obligatorios.
- [ ] Campos opcionales.
- [ ] Actualización completa mediante `PUT`.
- [ ] Actualización parcial mediante `PATCH`.

---

## Fase 6 — El enrutador y la API

Crear o completar el punto de entrada de la API.

- [ ] `index.php`.
- [ ] Definir las rutas CRUD para las seis tablas.
- [ ] Definir las rutas de listado.
- [ ] Definir las rutas de consulta individual.
- [ ] Definir las rutas `POST`.
- [ ] Definir las rutas `PUT`.
- [ ] Definir las rutas `PATCH`.
- [ ] Definir las rutas `DELETE`.
- [ ] Responder `405` cuando el método HTTP no esté permitido.
- [ ] Diferenciar un `404` de ruta inexistente de un `404` de registro no
      encontrado.
- [ ] Devolver respuestas JSON de forma uniforme.
- [ ] Respetar el contrato definido en `6_contratos.md`.

Las rutas deben mantenerse consistentes entre las seis entidades.

Ejemplo conceptual:

```http
GET    /api/area_conocimiento
GET    /api/area_conocimiento/{id}
POST   /api/area_conocimiento
PUT    /api/area_conocimiento/{id}
PATCH  /api/area_conocimiento/{id}
DELETE /api/area_conocimiento/{id}
```

El mismo patrón debe aplicarse a las otras cinco tablas.

**Verificación:**

Ejecutar los pasos correspondientes del:

```text
7_quickstart.md
```

La API debe poder realizar el CRUD completo de las seis tablas.

---

## Fase 7 — La prueba de capas

Crear y mantener una prueba de capas que permita comprobar la lógica sin
depender directamente de MariaDB.

- [ ] `pruebas/prueba_capas.php`.
- [ ] Crear un repositorio falso en memoria.
- [ ] El repositorio falso debe implementar la interfaz correspondiente.
- [ ] El falso debe permitir listar.
- [ ] El falso debe permitir consultar.
- [ ] El falso debe permitir crear.
- [ ] El falso debe permitir actualizar.
- [ ] El falso debe realizar borrado lógico, no borrado físico.
- [ ] El servicio debe poder probarse sin conectarse a MariaDB.
- [ ] Las reglas de negocio deben poder comprobarse de forma independiente
      de HTTP.

Ejecutar:

```powershell
docker compose exec <servicio-api> php pruebas/prueba_capas.php
```

La prueba debe comprobar como mínimo:

- [ ] Creación correcta.
- [ ] Consulta correcta.
- [ ] Actualización completa.
- [ ] Actualización parcial.
- [ ] Rechazo de datos inválidos.
- [ ] Registro inexistente.
- [ ] Borrado lógico.
- [ ] Un registro borrado lógicamente ya no aparece en las consultas normales.

---

## Fase 8 — LA PANTALLA: Frontend

La interfaz es la otra mitad de la versión v1.

Crear la estructura del Frontend separada de la API.

### Cliente de API

- [ ] `front_php/cliente_api.php`.
- [ ] Una función por operación necesaria.
- [ ] El cliente se comunica exclusivamente con la API.
- [ ] El Frontend no utiliza PDO.
- [ ] El Frontend no conoce las credenciales de MariaDB.
- [ ] El Frontend no ejecuta SQL.

### Entrada del Frontend

- [ ] `front_php/index.php`.
- [ ] Manejar correctamente las rutas de las pantallas.
- [ ] Evitar que el router PHP interfiera con archivos estáticos.
- [ ] Utilizar `return false` cuando corresponda para permitir servir archivos
      estáticos.

### Vistas

Crear las vistas necesarias:

- [ ] Marco general.
- [ ] Página de inicio.
- [ ] Listado de registros.
- [ ] Formulario de creación.
- [ ] Formulario de edición.
- [ ] Página 404.
- [ ] Mensajes de éxito.
- [ ] Mensajes de error.

### CRUD de las seis tablas

Implementar las pantallas para:

- [ ] `area_conocimiento`.
- [ ] `objetivo_desarrollo_sostenible`.
- [ ] `area_aplicacion`.
- [ ] `termino_clave`.
- [ ] `universidad`.
- [ ] `linea_investigacion`.

Cada entidad debe permitir desde el Frontend:

- [ ] Listar.
- [ ] Crear.
- [ ] Consultar.
- [ ] Editar.
- [ ] Eliminar lógicamente.
- [ ] Mostrar errores provenientes de la API.
- [ ] Confirmar operaciones exitosas.

### Bootstrap

- [ ] Utilizar Bootstrap para la interfaz.
- [ ] Bootstrap debe estar disponible localmente según la estrategia definida
      en el proyecto.
- [ ] No depender de un CDN para que la interfaz funcione.

### Formularios

- [ ] El formulario de creación envía los datos necesarios para `POST`.
- [ ] El formulario de edición permite actualización completa mediante
      `PUT` o actualización parcial mediante `PATCH`, según el diseño
      definido en `6_contratos.md`.
- [ ] Los botones y formularios no exponen datos sensibles.

**Verificación:**

Ejecutar:

```powershell
python pruebas_humo/humo_front.py
```

Además, realizar una comprobación manual de las seis entidades en el
navegador.

---

## Fase 9 — Pruebas de integración completas

Comprobar que API y Frontend funcionen juntos.

- [ ] Crear un registro desde el Frontend.
- [ ] Comprobar que aparece mediante la API.
- [ ] Consultar el registro desde el Frontend.
- [ ] Editarlo desde el Frontend.
- [ ] Comprobar el cambio mediante la API.
- [ ] Eliminarlo lógicamente desde el Frontend.
- [ ] Comprobar que deja de aparecer en los listados normales.
- [ ] Comprobar que el registro no fue eliminado físicamente de MariaDB.
- [ ] Repetir el flujo para las seis tablas.

También comprobar:

- [ ] Un registro inexistente devuelve `404`.
- [ ] Un cuerpo inválido devuelve `422`.
- [ ] Un método no permitido devuelve `405`.
- [ ] Un error interno devuelve `500`.
- [ ] Los mensajes mostrados por el Frontend corresponden a la respuesta
      de la API.

---

## Fase 10 — Datos de referencia

Antes de cerrar v1, comprobar que los datos de referencia del proyecto están
cargados correctamente.

- [ ] `area_conocimiento`: 218 registros.
- [ ] `objetivo_desarrollo_sostenible`: 17 registros.
- [ ] `area_aplicacion`: 21 registros.
- [ ] `universidad`: 6 registros.
- [ ] `termino_clave`: todos los registros entregados por la fuente de datos.
- [ ] `linea_investigacion`: todos los registros entregados por la fuente
      de datos.

Comprobar además:

- [ ] No existen duplicados donde la llave primaria no los permita.
- [ ] Los datos respetan los tipos definidos en el modelo.
- [ ] Las llaves primarias fueron cargadas correctamente.
- [ ] Los registros aparecen en el listado del Frontend.
- [ ] Los registros activos pueden consultarse mediante la API.

---

## Fase 11 — Seguridad y configuración

Antes de cerrar v1:

- [ ] No existen contraseñas de MariaDB en el código fuente.
- [ ] No existe ningún JWT implementado todavía: la autenticación corresponde
      a v3.
- [ ] No existen credenciales escritas en el Frontend.
- [ ] `.env` no está versionado.
- [ ] `.env.example` sí está versionado.
- [ ] `JWT_SECRET` queda reservado para v3.
- [ ] No existen tokens, contraseñas o claves privadas dentro del repositorio.
- [ ] Los errores no exponen credenciales ni información sensible.

---

## Fase 12 — Colección de pruebas

Crear o completar la colección de pruebas HTTP.

- [ ] Colección de Postman o herramienta equivalente.
- [ ] Prueba de listado de cada tabla.
- [ ] Prueba de consulta individual de cada tabla.
- [ ] Prueba de creación de cada tabla.
- [ ] Prueba de `PUT`.
- [ ] Prueba de `PATCH`.
- [ ] Prueba de `DELETE`.
- [ ] Prueba de registro inexistente.
- [ ] Prueba de datos inválidos.
- [ ] Prueba de método no permitido.
- [ ] Prueba de borrado lógico.
- [ ] Prueba de que los registros inactivos no aparecen en los listados.

El orden de ejecución debe seguir el flujo establecido en:

```text
7_quickstart.md
```

---

## Fase 13 — Cerrar v1

La versión v1 solamente se considera terminada cuando:

- [ ] Las seis tablas tienen CRUD completo en la API.
- [ ] Las seis tablas tienen CRUD completo en el Frontend.
- [ ] El borrado es lógico.
- [ ] Los registros inactivos no aparecen en consultas normales.
- [ ] Los datos de referencia fueron cargados.
- [ ] API y Frontend funcionan de extremo a extremo.
- [ ] La prueba de capas pasa correctamente.
- [ ] Las pruebas HTTP pasan correctamente.
- [ ] Las pruebas manuales del Frontend pasan correctamente.
- [ ] No existen secretos en el repositorio.
- [ ] `4_research.md` refleja las decisiones realmente implementadas.
- [ ] `5_data_model.md` coincide con la base de datos.
- [ ] `6_contratos.md` coincide con el comportamiento real de la API.
- [ ] `7_quickstart.md` permite reproducir la instalación y las pruebas.
- [ ] Este archivo `8_tasks.md` queda actualizado con las tareas realmente
      completadas.
- [ ] `9_checklist.md` es revisado y firmado por una persona.

### Criterio de cierre

La versión v1 se puede cerrar cuando una persona diferente al desarrollador
puede seguir `7_quickstart.md`, levantar el proyecto, comprobar la base de
datos, ejecutar las pruebas y realizar el CRUD de las seis tablas sin
necesitar modificar el código fuente.

Finalmente:

- [ ] Crear el commit final de v1.
- [ ] Revisar el Pull Request.
- [ ] Integrar únicamente mediante Pull Request.
- [ ] Crear el tag:

```text
v1
```

La versión v2 comienza únicamente después de cerrar correctamente v1.
