# Especificación — v1: Catálogos de Investigación

## 1. Alcance

La versión v1 del proyecto de investigación implementará el CRUD completo de seis tablas de catálogo que no dependen de otras tablas mediante llaves foráneas.

Las tablas incluidas son:

1. `area_conocimiento`
2. `objetivo_desarrollo_sostenible`
3. `area_aplicacion`
4. `termino_clave`
5. `universidad`
6. `linea_investigacion`

La solución estará compuesta por:

- API desarrollada en PHP.
- Frontend desarrollado en PHP.
- Base de datos MariaDB.
- Interfaz utilizando Bootstrap.
- Programación Orientada a Objetos (P.O.O.).
- Arquitectura por capas.
- API REST.
- Eliminación lógica de registros.
- Variables sensibles almacenadas mediante variables de entorno.
- Pruebas de los componentes principales.

La versión v1 debe permitir administrar completamente los seis catálogos desde el frontend mediante la API.

---

## 2. Fuera de alcance

En la versión v1 **NO** se implementarán:

- Autenticación de usuarios.
- Login.
- JWT.
- Roles y permisos.
- Middleware de autenticación.
- Administración de usuarios.
- Administración de roles.
- Consultas multi-tabla.
- Dashboard estadístico.
- Gráficas.
- Páginas corporativas.
- PWA.
- Publicación en servidor.
- Tablas que dependan de llaves foráneas.

Las funcionalidades anteriores serán desarrolladas en las versiones posteriores del proyecto.

---

## 3. Tablas incluidas en v1

### 3.1 `area_conocimiento`

Tabla utilizada para almacenar las áreas del conocimiento.

#### Campos

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `id` | INT | Sí | Identificador del área de conocimiento |
| `gran_area` | VARCHAR(60) | Sí | Gran área de conocimiento |
| `area` | VARCHAR(60) | Sí | Área de conocimiento |
| `disciplina` | VARCHAR(60) | Sí | Disciplina asociada |

#### Llave primaria

```text
id
```

### 3.2 `objetivo_desarrollo_sostenible`

Tabla utilizada para almacenar los Objetivos de Desarrollo Sostenible.

#### Campos

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `id` | INT | Sí | Identificador del ODS |
| `nombre` | VARCHAR(60) | Sí | Nombre del objetivo |
| `categoria` | VARCHAR(45) | Sí | Categoría del ODS |

#### Llave primaria

```text
id
```

#### Datos esperados

La tabla debe contener los 17 Objetivos de Desarrollo Sostenible.

Las categorías contempladas son:

- Social
- Económica
- Ambiental

### 3.3 `area_aplicacion`

Tabla utilizada para almacenar las áreas o sectores de aplicación de la investigación.

#### Campos

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `id` | INT | Sí | Identificador del área |
| `nombre` | VARCHAR(60) | Sí | Nombre del área de aplicación |

#### Llave primaria

```text
id
```

#### Datos esperados

Se deben cargar los datos de referencia proporcionados para el proyecto.

La cantidad indicada para la versión v1 es de **21 registros**.

### 3.4 `termino_clave`

Tabla utilizada para almacenar términos clave relacionados con la investigación.

#### Campos

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `termino` | VARCHAR(30) | Sí | Término clave |
| `termino_ingles` | VARCHAR(30) | No | Traducción del término al inglés |

#### Llave primaria

```text
termino
```

### 3.5 `universidad`

Tabla utilizada para almacenar las universidades o instituciones relacionadas con la investigación.

#### Campos

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `id` | INT | Sí | Identificador de la universidad |
| `nombre` | VARCHAR(60) | Sí | Nombre de la universidad |
| `tipo` | VARCHAR(45) | Sí | Tipo de institución |
| `ciudad` | VARCHAR(45) | Sí | Ciudad de ubicación |

#### Llave primaria

```text
id
```

#### Datos esperados

Se deben cargar los datos de referencia proporcionados para el proyecto.

La cantidad indicada para la versión v1 es de **6 registros**.

### 3.6 `linea_investigacion`

Tabla utilizada para almacenar las líneas de investigación.

#### Campos

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `id` | INT | Sí | Identificador de la línea |
| `nombre` | VARCHAR(45) | Sí | Nombre de la línea |
| `descripcion` | VARCHAR(256) | Sí | Descripción de la línea |

#### Llave primaria

```text
id
```

#### Característica especial

El campo `id` utiliza incremento automático.

---

## 4. Requisitos funcionales de la API

Cada una de las seis tablas debe contar con las operaciones CRUD correspondientes.

### RF1 — Consultar registros

El sistema debe permitir consultar los registros de cada tabla.

**Método**

```text
GET
```

**Ejemplo**

```http
GET /api/area-conocimiento
```

**Resultado esperado**

La API debe devolver una respuesta JSON con los registros disponibles.

Ejemplo:

```json
{
    "success": true,
    "data": []
}
```

Los registros marcados como inactivos no deben aparecer en las consultas normales.

### RF2 — Consultar un registro

El sistema debe permitir consultar un registro específico mediante su identificador.

**Método**

```text
GET
```

**Ejemplo**

```http
GET /api/area-conocimiento/1
```

Para tablas cuya llave primaria no sea un entero, como `termino_clave`, se debe utilizar el valor correspondiente de la llave primaria.

**Resultado esperado**

Si el registro existe, la API debe devolverlo en formato JSON.

Si el registro no existe, debe devolver un error apropiado.

### RF3 — Crear un registro

El sistema debe permitir crear nuevos registros.

**Método**

```text
POST
```

**Ejemplo**

```http
POST /api/area-conocimiento
```

**Body de ejemplo**

```json
{
    "id": 1,
    "gran_area": "Ciencias Naturales",
    "area": "Ciencias Biológicas",
    "disciplina": "Biología"
}
```

**Reglas**

- Los campos obligatorios deben ser validados.
- No se deben permitir registros duplicados.
- Los tipos de datos deben coincidir con la estructura de la base de datos.
- Los errores deben ser informados mediante respuestas JSON.

### RF4 — Actualizar completamente un registro

El sistema debe permitir modificar completamente un registro.

**Método**

```text
PUT
```

**Ejemplo**

```http
PUT /api/area-conocimiento/1
```

**Reglas**

- Se deben validar los campos obligatorios.
- Los datos enviados deben reemplazar los valores actuales.
- El registro debe existir antes de realizar la actualización.

### RF5 — Actualizar parcialmente un registro

El sistema debe permitir modificar parcialmente un registro.

**Método**

```text
PATCH
```

**Ejemplo**

```http
PATCH /api/area-conocimiento/1
```

**Body de ejemplo**

```json
{
    "disciplina": "Nueva disciplina"
}
```

**Reglas**

- Solo se deben modificar los campos enviados.
- Los campos no enviados deben conservar su valor.
- El registro debe existir.
- Los datos deben ser validados.

### RF6 — Eliminación lógica

El sistema debe permitir desactivar registros sin eliminarlos físicamente de la base de datos.

**Método**

```text
DELETE
```

**Ejemplo**

```http
DELETE /api/area-conocimiento/1
```

La eliminación será lógica.

El registro no debe desaparecer físicamente de la base de datos.

Debe quedar marcado como inactivo y las consultas normales no deben mostrar registros inactivos.

### RF7 — Diagnóstico de la API

La API debe disponer de un endpoint de diagnóstico que permita verificar que el sistema está funcionando correctamente.

**Método**

```text
GET
```

**Ejemplo**

```http
GET /api/
```

**Resultado esperado**

```json
{
    "success": true,
    "message": "API funcionando correctamente"
}
```

---

## 5. Requisitos del frontend

El frontend debe permitir administrar las seis tablas incluidas en la versión v1.

Para cada tabla se debe disponer de una interfaz que permita:

- Consultar registros.
- Crear registros.
- Editar registros.
- Actualizar parcialmente cuando corresponda.
- Desactivar registros.
- Visualizar mensajes de éxito.
- Visualizar mensajes de error.
- Validar los campos del formulario.

---

## 6. Interfaz gráfica

El frontend debe utilizar Bootstrap.

La interfaz debe ser clara, organizada y responsive.

Cada módulo CRUD debe incluir como mínimo:

- Tabla de registros.
- Botón para crear.
- Botón para editar.
- Botón para eliminar/desactivar.
- Formulario correspondiente.
- Mensajes de validación.
- Mensajes de operación exitosa.
- Mensajes de error.

---

## 7. Formularios

Los formularios deben utilizar controles HTML adecuados para cada tipo de información.

Ejemplos:

- `text` para nombres.
- `number` para identificadores numéricos.
- `date` para fechas cuando sean necesarias.
- `textarea` para descripciones.
- `select` cuando posteriormente existan relaciones entre tablas.

Los campos obligatorios deben identificarse claramente.

---

## 8. Validaciones

La validación debe realizarse tanto en frontend como en backend.

### Validaciones generales

El sistema debe comprobar:

- Campos obligatorios.
- Longitud máxima.
- Tipo de dato.
- Identificadores válidos.
- Registros duplicados.
- Existencia del registro antes de actualizarlo.
- Existencia del registro antes de desactivarlo.

> [!IMPORTANT]
> El backend nunca debe confiar únicamente en las validaciones realizadas por el frontend.

---

## 9. Arquitectura

El proyecto debe utilizar Programación Orientada a Objetos.

La arquitectura debe separar las responsabilidades principales.

Se propone una estructura similar a:

```text
API/
├── config/
├── controllers/
├── services/
├── repositories/
├── models/
├── interfaces/
├── routes/
├── database/
├── tests/
└── public/
```

Y para el frontend:

```text
Frontend/
├── config/
├── controllers/
├── services/
├── views/
├── assets/
├── public/
└── tests/
```

La estructura definitiva será especificada en el documento `3_plan.md`.

---

## 10. Modelos

Cada tabla de la versión v1 debe tener una representación mediante una clase de modelo.

Los modelos deben representar los datos correspondientes a cada entidad.

Ejemplo conceptual:

```php
class AreaConocimiento
{
    private int $id;
    private string $granArea;
    private string $area;
    private string $disciplina;
}
```

La implementación definitiva de las clases se definirá durante la etapa de planificación y desarrollo.

---

## 11. Repositorios

El acceso a la base de datos debe estar separado de la lógica de negocio.

Cada entidad debe disponer de un repositorio encargado de realizar operaciones como:

```text
findAll()
findById()
create()
update()
patch()
delete()
```

Los repositorios utilizarán PDO para comunicarse con MariaDB.

---

## 12. Servicios

La lógica de negocio debe ubicarse en clases de servicio.

Los servicios serán responsables de:

- Validar reglas de negocio.
- Coordinar repositorios.
- Procesar solicitudes.
- Controlar errores.
- Preparar las respuestas correspondientes.

Los controladores no deben contener toda la lógica de negocio.

---

## 13. Controladores

Los controladores deben recibir las solicitudes HTTP y comunicarse con los servicios correspondientes.

Ejemplo conceptual:

```text
Request
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
MariaDB
```

La respuesta debe regresar mediante JSON en el caso de la API.

---

## 14. Interfaces

Cuando corresponda, los repositorios y servicios deben utilizar interfaces para separar contratos de implementación.

Ejemplo:

```php
interface AreaConocimientoRepositoryInterface
{
    public function findAll(): array;

    public function findById(int $id): ?AreaConocimiento;

    public function create(AreaConocimiento $area): bool;

    public function update(int $id, AreaConocimiento $area): bool;

    public function patch(int $id, array $data): bool;

    public function delete(int $id): bool;
}
```

Esto permitirá realizar pruebas utilizando implementaciones falsas o simuladas.

---

## 15. Base de datos

La base de datos utilizada será **MariaDB**.

El nombre de la base de datos será:

```text
investigacion
```

La conexión debe realizarse utilizando variables de entorno.

No se deben almacenar directamente en el código:

- Usuario de base de datos.
- Contraseña.
- Host sensible.
- Puerto sensible.
- `JWT_SECRET` para versiones posteriores.

---

## 16. Variables de entorno

Las variables sensibles deben almacenarse mediante un archivo `.env`.

El archivo `.env` debe estar incluido en `.gitignore`.

El repositorio debe contener un archivo:

```text
.env.example
```

Este archivo debe mostrar únicamente los nombres de las variables necesarias, sin valores secretos reales.

Ejemplo:

```env
DB_HOST=
DB_PORT=
DB_DATABASE=investigacion
DB_USERNAME=
DB_PASSWORD=
```

---

## 17. Seguridad

No se deben subir secretos al repositorio.

No se deben incluir:

- Contraseñas reales.
- Tokens.
- Claves privadas.
- Credenciales de bases de datos.
- `JWT_SECRET` real.

Las credenciales deben manejarse mediante variables de entorno.

---

## 18. Respuestas de la API

Las respuestas de la API deben utilizar JSON.

Se recomienda mantener una estructura consistente.

### Respuesta exitosa

```json
{
    "success": true,
    "data": []
}
```

### Respuesta de error

```json
{
    "success": false,
    "message": "Descripción del error"
}
```

Los códigos HTTP deben utilizarse de acuerdo con el resultado de la operación.

Ejemplos:

```text
200 OK
201 Created
400 Bad Request
404 Not Found
409 Conflict
500 Internal Server Error
```

---

## 19. Manejo de errores

El sistema debe manejar errores de manera controlada.

No se deben mostrar al usuario final errores internos de PHP o SQL.

Los errores deben registrarse cuando sea necesario y la API debe devolver mensajes comprensibles.

Ejemplo:

```json
{
    "success": false,
    "message": "El registro solicitado no existe"
}
```

---

## 20. Pruebas

Se deben realizar pruebas de las operaciones principales.

Como mínimo se deben probar:

- Consulta de registros.
- Consulta de un registro.
- Creación.
- Actualización.
- Actualización parcial.
- Eliminación lógica.
- Consulta de registro inexistente.
- Creación de registro duplicado.
- Validación de campos obligatorios.

La arquitectura debe permitir utilizar repositorios falsos o simulados para las pruebas.

---

## 21. Datos de referencia

La versión v1 debe utilizar los datos de referencia proporcionados para el proyecto.

Datos conocidos:

| Tabla | Cantidad indicada |
|---|---|
| `area_conocimiento` | 218 |
| `objetivo_desarrollo_sostenible` | 17 |
| `area_aplicacion` | 21 |
| `termino_clave` | No especificada |
| `universidad` | 6 |
| `linea_investigacion` | No especificada |

Los datos deben cargarse correctamente en MariaDB.

---

## 22. Criterios de aceptación de la API

La versión v1 será aceptada cuando:

- [ ] La base de datos `investigacion` esté creada.
- [ ] Las seis tablas estén disponibles.
- [ ] La API pueda consultar los seis catálogos.
- [ ] La API pueda consultar un registro individual.
- [ ] La API pueda crear registros.
- [ ] La API pueda actualizar registros mediante `PUT`.
- [ ] La API pueda actualizar registros mediante `PATCH`.
- [ ] La API pueda realizar eliminación lógica.
- [ ] La API filtre los registros inactivos.
- [ ] Exista un endpoint de diagnóstico.
- [ ] Las respuestas sean JSON.
- [ ] Los errores utilicen códigos HTTP adecuados.
- [ ] Los datos de referencia estén cargados.
- [ ] No existan secretos en el repositorio.

---

## 23. Criterios de aceptación del frontend

La versión v1 será aceptada cuando:

- [ ] Exista una interfaz para cada una de las seis tablas.
- [ ] Se puedan listar registros.
- [ ] Se puedan crear registros.
- [ ] Se puedan editar registros.
- [ ] Se puedan desactivar registros.
- [ ] Se muestren mensajes de éxito.
- [ ] Se muestren mensajes de error.
- [ ] Los formularios tengan validaciones.
- [ ] Se utilice Bootstrap.
- [ ] La interfaz sea responsive.
- [ ] El frontend consuma la API correctamente.

---

## 24. Criterios de aceptación de arquitectura

La solución debe:

- [ ] Utilizar PHP.
- [ ] Utilizar Programación Orientada a Objetos.
- [ ] Utilizar MariaDB.
- [ ] Utilizar PDO.
- [ ] Separar controladores, servicios y repositorios.
- [ ] Utilizar interfaces donde corresponda.
- [ ] Mantener separada la lógica de acceso a datos.
- [ ] Permitir realizar pruebas mediante repositorios falsos o simulados.
- [ ] No utilizar un framework backend en v1.
- [ ] Mantener el código organizado y documentado.

---

## 25. Criterios de aceptación de Git

El proyecto debe cumplir las reglas establecidas para el desarrollo.

Se deben utilizar dos repositorios privados:

- API
- Frontend

El profesor:

```text
ccastro2050
```

debe estar invitado desde el inicio.

- No se debe trabajar directamente sobre `main`.
- Cada integrante debe trabajar sobre su propia rama.
- Los cambios deben integrarse mediante Pull Request.
- Solamente el responsable de `main` debe realizar el merge.

---

## 26. Flujo de desarrollo

El desarrollo seguirá el enfoque SDD por versiones.

El flujo esperado será:

```text
Spec
  ↓
Plan
  ↓
Tasks
  ↓
Implementación
  ↓
Pruebas
  ↓
Pull Request
  ↓
Merge
  ↓
Tag de versión
```

No se debe iniciar una versión sin tener previamente definida su especificación.

---

## 27. Alcance de versiones posteriores

### v2

Se implementarán las diez tablas restantes relacionadas con investigación y sus llaves foráneas.

Entre ellas:

- `docente`
- `grupo_investigacion`
- `semillero`
- `participa_grupo`
- `participa_semillero`
- `grupo_linea`
- `semillero_linea`
- `ac_linea`
- `ods_linea`
- `aa_linea`

Se deberá garantizar la integridad referencial.

Los campos que representen relaciones deberán utilizar información obtenida mediante la API.

El semillero deberá asociarse a un `grupo_investigacion` existente.

---

## 28. v3

Se implementará:

- `POST /api/login`
- Autenticación mediante JWT.
- Middleware de autenticación.
- Autorización por roles.
- Menú dinámico según rol.
- Administración de usuarios.
- Administración de roles.
- Administración de relaciones usuario-rol.
- Logout.

El usuario con rol de consulta no podrá realizar operaciones de escritura.

Solo los administradores podrán administrar usuarios y roles.

---

## 29. v4

Se implementará:

- Diez consultas que involucren cuatro o más tablas.
- Dashboard.
- Gráficas.
- Páginas corporativas.
- Imagen propia.
- Diseño responsive.
- PWA.
- Publicación del sistema.

La versión final deberá encontrarse publicada y funcionando correctamente con las variables de entorno configuradas en el servidor.

---

## 30. Cierre de v1

La versión v1 se considerará terminada cuando:

- [ ] Los seis CRUD estén completamente implementados.
- [ ] La API funcione correctamente.
- [ ] El frontend consuma la API.
- [ ] Bootstrap esté integrado.
- [ ] Los datos de referencia estén cargados.
- [ ] La eliminación lógica funcione.
- [ ] Los registros inactivos no aparezcan en consultas normales.
- [ ] Las validaciones funcionen.
- [ ] Las pruebas principales estén realizadas.
- [ ] No existan secretos en Git.
- [ ] La documentación correspondiente esté actualizada.
- [ ] El código esté integrado mediante Pull Request.
- [ ] Se cree el tag:

```text
v1
```

---

## 31. Clarificaciones pendientes

Antes de comenzar la implementación definitiva se deben revisar y confirmar los siguientes puntos:

- La especificación utiliza como fuente principal el SQL oficial entregado para el proyecto.
- El ejemplo utilizado en clase puede tener diferencias respecto al SQL oficial; cuando exista una diferencia, debe prevalecer el requerimiento oficial del proyecto.
- Se debe confirmar cómo se implementará el campo de eliminación lógica (`activo`) en las seis tablas de v1, debido a que el SQL proporcionado actualmente no lo incluye en dichas tablas.
- Se debe confirmar si las tablas de v1 deben modificarse para incluir dicho campo.
- Se debe confirmar la estructura final de las rutas de la API.
- Se debe confirmar la estructura final de carpetas de API y Frontend durante el documento `3_plan.md`.
- Se debe confirmar la estrategia exacta de pruebas durante el documento de planificación.

---

## 32. Resultado esperado

Al finalizar la versión v1, el proyecto deberá permitir administrar los seis catálogos principales de investigación mediante una aplicación web desarrollada en PHP, con frontend Bootstrap, API REST, base de datos MariaDB, arquitectura orientada a objetos y separación de responsabilidades.

La implementación deberá quedar preparada para continuar posteriormente con las versiones:

```text
v1 → CRUD de 6 tablas
v2 → CRUD de 10 tablas relacionadas
v3 → Autenticación y autorización
v4 → Consultas, dashboard y publicación
```

El cierre de esta especificación habilita la elaboración del documento:

```text
3_plan.md
```

que definirá la arquitectura técnica, estructura de carpetas, clases, interfaces, controladores, servicios, repositorios, rutas y estrategia de implementación.
