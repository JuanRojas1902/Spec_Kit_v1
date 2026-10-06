# Research — v1: Investigación

## Decisiones técnicas y de diseño

Este documento registra las decisiones tomadas para la versión 1 del proyecto de investigación.

La finalidad es dejar documentado qué alternativas fueron consideradas, cuál fue la decisión adoptada, por qué se tomó y cuáles son sus consecuencias para el desarrollo.

---

## D-v1-1 — PHP puro, sin framework y sin Composer

### Alternativas

- Utilizar PHP puro.
- Utilizar un framework como Laravel o Symfony.
- Utilizar Composer y diferentes paquetes externos.

### Decisión

Para la versión 1 se utilizará **PHP puro**, sin framework de backend y sin Composer.

La API será desarrollada utilizando las características nativas de PHP, PDO para la conexión con MariaDB y una arquitectura organizada por capas.

### Consecuencias

- El equipo deberá implementar manualmente parte de la estructura de la API.
- Se tendrá mayor control sobre el funcionamiento interno del proyecto.
- Se evita agregar dependencias externas innecesarias.
- La estructura deberá mantenerse organizada para facilitar las siguientes versiones.

**Estado: vigente**

---

## D-v1-2 — MariaDB como motor de base de datos

### Alternativas

- MariaDB.
- MySQL.
- PostgreSQL.
- SQLite.

### Decisión

Se utilizará **MariaDB** como motor de base de datos del proyecto.

La base de datos se llamará `investigacion`.

### Consecuencias

- Las consultas SQL deberán ser compatibles con MariaDB.
- La conexión desde PHP se realizará mediante PDO.
- Las tablas utilizarán InnoDB para soportar integridad referencial en las versiones posteriores.
- Las decisiones relacionadas con SQL deberán considerar el comportamiento específico de MariaDB.

**Estado: vigente**

---

## D-v1-3 — Seis tablas iniciales para la versión 1

### Alternativas

- Implementar todas las tablas de la base de datos desde la primera versión.
- Implementar únicamente las tablas necesarias para la v1.
- Comenzar únicamente con una tabla y dejar las demás para versiones posteriores.

### Decisión

La versión 1 implementará CRUD completo para las siguientes seis tablas:

1. `area_conocimiento`
2. `objetivo_desarrollo_sostenible`
3. `area_aplicacion`
4. `termino_clave`
5. `universidad`
6. `linea_investigacion`

Estas tablas se implementarán sin agregar relaciones nuevas entre ellas en esta versión, manteniendo el alcance definido para v1.

### Consecuencias

- El trabajo de la v1 queda limitado a seis CRUD.
- Las tablas relacionadas y las claves foráneas correspondientes se trabajarán en la v2.
- La arquitectura deberá permitir agregar nuevas entidades posteriormente sin tener que reconstruir la API.

**Estado: vigente**

---

## D-v1-4 — Eliminación lógica mediante `activo`

### Alternativas

- Eliminar físicamente los registros mediante `DELETE`.
- Utilizar eliminación lógica mediante un campo `activo`.
- No permitir eliminación de registros.

### Decisión

Se utilizará **eliminación lógica**.

Las seis tablas de la v1 deberán contar con un campo:

```sql
activo TINYINT(1) NOT NULL DEFAULT 1


--------------------------------------------------
Cuando un registro sea eliminado desde la aplicación, no se ejecutará un DELETE físico. En su lugar, el campo activo cambiará de 1 a 0.

Esta decisión representa un ajuste respecto al SQL inicial proporcionado para las tablas de la v1, ya que dicho SQL no incluye originalmente el campo activo.

Consecuencias
Los datos eliminados físicamente no se perderán.
Los registros con activo = 0 deberán quedar excluidos de las consultas normales.
Los endpoints de eliminación realizarán una actualización lógica.
El esquema SQL deberá ser actualizado antes de completar la implementación de la v1.
La documentación deberá reflejar este cambio respecto al SQL inicial.

Estado: vigente

D-v1-5 — Filtrado obligatorio de registros inactivos
Alternativas
Mostrar todos los registros independientemente de su estado.
Filtrar registros inactivos únicamente en el frontend.
Filtrar registros inactivos desde las consultas del backend.
Decisión

Todas las consultas normales de lectura deberán considerar el estado lógico del registro.

Para las tablas de la v1, las consultas de listado y búsqueda deberán utilizar una condición equivalente a:

WHERE activo = 1

El filtro se realizará principalmente en el backend y no se confiará únicamente en el frontend.

Consecuencias
Los registros eliminados lógicamente no aparecerán en los listados normales.
El comportamiento será consistente independientemente del cliente que consuma la API.
Los repositorios deberán tener presente el estado activo en sus consultas.
Si en el futuro se necesita consultar registros inactivos, deberá existir una operación específica para ello.

Estado: vigente

D-v1-6 — Arquitectura organizada por capas
Alternativas
Colocar toda la lógica en los archivos de los endpoints.
Utilizar una arquitectura por capas.
Utilizar una arquitectura basada en un framework.
Decisión

La API se organizará mediante capas con responsabilidades separadas:

Controller
    ↓
Service
    ↓
Repository
    ↓
Database

Además, se utilizarán modelos e interfaces cuando sean necesarios para mantener separadas las responsabilidades.

Consecuencias
Los Controllers manejarán principalmente HTTP.
Los Services contendrán la lógica de negocio.
Los Repositories manejarán las consultas SQL.
La conexión con MariaDB quedará centralizada.
El código será más fácil de probar y mantener.
Las siguientes versiones podrán reutilizar la estructura creada en v1.

Estado: vigente

D-v1-7 — El Service no conoce HTTP
Alternativas
Permitir que el Service maneje directamente $_GET, $_POST y códigos HTTP.
Mantener HTTP únicamente en el Controller.
Colocar toda la lógica en el Repository.
Decisión

El Service no tendrá conocimiento de HTTP.

El Service recibirá datos ya procesados por el Controller y devolverá resultados o errores de negocio.

No deberá utilizar directamente:

$_GET
$_POST
$_PUT
http_response_code()
header()
Consecuencias
El Service podrá probarse sin necesidad de realizar solicitudes HTTP.
La lógica de negocio quedará desacoplada del transporte.
El Controller será responsable de convertir la respuesta del Service en una respuesta HTTP.
La arquitectura será más fácil de extender y mantener.

Estado: vigente

D-v1-8 — El Repository es responsable del SQL
Alternativas
Escribir SQL directamente dentro del Controller.
Escribir SQL dentro del Service.
Centralizar las consultas SQL en los Repositories.
Decisión

Los Repositories serán responsables del acceso a datos y de las consultas SQL.

El Service no construirá directamente consultas SQL.

Ejemplo conceptual:

Controller
    ↓
Service
    ↓
AreaConocimientoRepository
    ↓
MariaDB
Consecuencias
Las consultas SQL estarán centralizadas.
Será más sencillo modificar una consulta sin afectar al Controller.
El Service podrá trabajar mediante interfaces de Repository.
Se facilita la creación de repositorios falsos para pruebas.

Estado: vigente

D-v1-9 — PDO y consultas preparadas
Alternativas
Utilizar mysqli.
Utilizar PDO.
Construir consultas concatenando valores recibidos del usuario.
Decisión

La conexión a MariaDB se realizará mediante PDO.

Las consultas que reciban valores externos deberán utilizar sentencias preparadas.

Ejemplo:

$stmt = $pdo->prepare(
    'SELECT *
     FROM area_conocimiento
     WHERE id = :id
       AND activo = 1'
);

$stmt->execute([
    ':id' => $id
]);

No se permitirá concatenar directamente valores recibidos del cliente dentro de las consultas SQL.

Consecuencias
Se reduce el riesgo de SQL Injection.
La conexión y las consultas quedan estandarizadas.
Los Repositories utilizarán una estrategia común de acceso a datos.
Se deberá prestar atención a los tipos de parámetros utilizados por PDO y MariaDB.

Estado: vigente

D-v1-10 — Diferencia entre PUT y PATCH
Alternativas
Utilizar solamente POST para crear y actualizar.
Utilizar PUT para cualquier actualización.
Utilizar PUT y PATCH según el tipo de actualización.
Decisión

Se utilizarán los métodos HTTP de acuerdo con su propósito:

POST: creación de recursos.
GET: consulta de recursos.
PUT: reemplazo completo de un recurso.
PATCH: actualización parcial de un recurso.
DELETE: eliminación lógica del recurso.

En la implementación se priorizará un contrato consistente entre los endpoints.

Consecuencias
El frontend deberá utilizar el método HTTP correspondiente.
El Controller deberá validar correctamente los datos recibidos según la operación.
Las actualizaciones parciales no deberán obligar al cliente a enviar información que no desea modificar.

Estado: vigente

D-v1-11 — Formato uniforme de errores de la API
Alternativas
Devolver errores con formatos diferentes dependiendo del endpoint.
Devolver únicamente mensajes de texto.
Utilizar un formato JSON uniforme.
Decisión

La API utilizará un formato JSON uniforme para comunicar errores.

La estructura base será:

{
    "estado": "error",
    "mensaje": "Descripción general del error",
    "detalle": "Información adicional",
    "errores": []
}

Cuando sea necesario comunicar errores específicos de validación, el arreglo errores podrá contener información adicional.

Consecuencias
El frontend podrá manejar errores de forma uniforme.
Los Controllers deberán respetar el contrato de respuesta.
Los mensajes técnicos no deberán exponerse innecesariamente al usuario final.
Será más sencillo depurar las respuestas de la API.

Estado: vigente

D-v1-12 — Frontend separado del API
Alternativas
Crear frontend y backend dentro de los mismos archivos.
Crear un frontend independiente que consuma la API.
Utilizar un framework frontend.
Decisión

El frontend será un proyecto separado del API.

El frontend consumirá los endpoints HTTP de la API y no accederá directamente a MariaDB.

La comunicación seguirá el flujo:

Frontend
    ↓ HTTP/JSON
API
    ↓ PDO
MariaDB
Consecuencias
El frontend no tendrá credenciales de la base de datos.
La API será responsable de la lógica y acceso a datos.
Será posible modificar el frontend sin modificar directamente la estructura interna de la API.
Se podrá reutilizar la API posteriormente para otros clientes.

Estado: vigente

D-v1-13 — Bootstrap para la interfaz
Alternativas
Crear todo el CSS manualmente.
Utilizar Bootstrap.
Utilizar otro framework CSS.
Decisión

El frontend utilizará Bootstrap para construir la interfaz de usuario.

Bootstrap se utilizará principalmente para:

formularios;
tablas;
botones;
navegación;
alertas;
modales;
diseño responsive.
Consecuencias
Se reduce el tiempo necesario para crear estilos básicos.
La interfaz tendrá componentes reutilizables.
El frontend deberá mantener una estructura clara para facilitar las futuras versiones.
Se podrá personalizar Bootstrap cuando sea necesario para cumplir con el diseño del proyecto.

Estado: vigente

D-v1-14 — Separación de responsabilidades entre listados y formularios
Alternativas
Crear una única pantalla que contenga toda la lógica.
Separar visualmente listado y formulario.
Crear una pantalla independiente para cada operación.
Decisión

Cada CRUD tendrá una estructura clara donde el usuario pueda:

visualizar los registros activos;
crear un registro;
editar un registro;
realizar la eliminación lógica;
recibir mensajes de éxito o error.

El listado será responsable de mostrar los datos disponibles y las acciones correspondientes.

El formulario será responsable de capturar y validar los datos necesarios para crear o actualizar un registro.

Consecuencias
La interfaz será más fácil de utilizar.
Las responsabilidades del código frontend estarán mejor separadas.
Se podrán reutilizar componentes entre los seis CRUD.
La implementación de nuevas tablas en v2 será más sencilla.

Estado: vigente

D-v1-15 — termino_clave mantiene termino como clave primaria
Alternativas
Agregar un id numérico a termino_clave.
Mantener termino como clave primaria.
Crear una clave compuesta.
Decisión

Se mantendrá la estructura definida en el modelo inicial:

CREATE TABLE termino_clave (
    termino VARCHAR(30) NOT NULL,
    termino_ingles VARCHAR(30),
    PRIMARY KEY (termino)
);

Por lo tanto, termino será utilizado como identificador del registro.

Consecuencias
No será necesario agregar un identificador numérico artificial.
El frontend deberá manejar correctamente valores de texto como identificadores.
Las rutas del API deberán contemplar la codificación de valores que contengan espacios o caracteres especiales.
Las operaciones de actualización y eliminación deberán identificar el registro mediante termino.

Estado: vigente

D-v1-16 — Carga de datos de referencia del Excel
Alternativas
Crear las tablas vacías y continuar sin datos.
Introducir manualmente algunos registros.
Cargar los datos de referencia proporcionados en Excel.
Decisión

Los datos de referencia proporcionados para el proyecto deberán cargarse en la base de datos antes del cierre de la v1.

Entre los datos conocidos se encuentran:

area_conocimiento: 218 registros.
objetivo_desarrollo_sostenible: 17 registros.
area_aplicacion: 21 registros.
universidad: 6 registros.

Para las tablas cuyo número de registros no se haya especificado, se deberá utilizar la información oficial suministrada para el proyecto.

Consecuencias
Los CRUD podrán probarse con datos reales de referencia.
Las pruebas del frontend serán más representativas.
Se podrá comprobar que los formularios y listados funcionan con datos reales.
La carga de datos deberá hacerse de forma controlada para evitar duplicados o registros inconsistentes.

Estado: vigente

D-v1-17 — Credenciales mediante variables de entorno
Alternativas
Escribir las credenciales directamente en PHP.
Guardar las credenciales en un archivo versionado.
Utilizar variables de entorno.
Decisión

Las credenciales de conexión a MariaDB y otros secretos del sistema se manejarán mediante variables de entorno.

Se utilizará un archivo .env de manera local cuando corresponda, pero este archivo no será incluido en Git.

El repositorio tendrá un archivo:

.env.example

con los nombres de las variables necesarias, pero sin secretos reales.

Ejemplo conceptual:

DB_HOST=
DB_NAME=investigacion
DB_USER=
DB_PASSWORD=
JWT_SECRET=

El secreto JWT_SECRET será utilizado posteriormente en v3.

Consecuencias
No se expondrán credenciales en el repositorio.
Cada entorno podrá utilizar sus propias credenciales.
.env deberá estar incluido en .gitignore.
La configuración de producción deberá proporcionar las variables mediante el entorno del servidor.

Estado: vigente

D-v1-18 — Pruebas del Service mediante repositorios falsos
Alternativas
Probar solamente mediante navegador.
Probar solamente los endpoints HTTP.
Probar la lógica de negocio de manera independiente.
Decisión

La lógica principal de los Services deberá poder probarse sin depender directamente de MariaDB.

Para ello, los Services dependerán de interfaces de Repository cuando sea necesario.

Esto permitirá utilizar repositorios falsos o mocks para probar escenarios como:

creación correcta;
actualización;
registro inexistente;
validación;
eliminación lógica;
errores de negocio.
Consecuencias
Las pruebas serán más rápidas.
Los errores de lógica podrán identificarse antes de probar todo el flujo HTTP.
La arquitectura por capas tendrá una utilidad práctica.
Las interfaces deberán mantenerse simples y claras.

Estado: vigente

D-v1-19 — Autenticación y autorización se implementan en v3
Alternativas
Implementar login y JWT desde v1.
Implementar una autenticación básica temporal.
Dejar autenticación y autorización para una versión posterior.
Decisión

La versión 1 no implementará autenticación ni autorización.

El sistema de login mediante:

POST /api/login

y el uso de JWT se implementarán en v3, junto con:

middleware de autenticación;
autorización por roles;
usuarios;
roles;
relación usuario-rol;
restricciones para usuarios de consulta.
Consecuencias
Los CRUD de v1 podrán desarrollarse sin agregar complejidad de autenticación.
La arquitectura deberá permitir incorporar middleware posteriormente.
No se deberán diseñar soluciones temporales de autenticación que después tengan que eliminarse.
La seguridad de autenticación será una responsabilidad específica de v3.

Estado: vigente

D-v1-20 — Relaciones y claves foráneas se implementan en v2
Alternativas
Implementar todas las relaciones desde v1.
Crear primero los CRUD independientes.
Implementar las relaciones en una segunda versión.
Decisión

Las relaciones entre las entidades que corresponden a la segunda etapa se implementarán en v2.

En v2 se incorporarán los CRUD de las tablas restantes y las relaciones mediante claves foráneas.

Los campos que dependan de otras tablas deberán convertirse en controles adecuados del frontend, principalmente listas desplegables cargadas mediante la API.

Consecuencias
La v1 puede concentrarse en CRUD independientes.
La v2 incorporará progresivamente la complejidad relacional.
El frontend deberá prepararse para consumir catálogos desde la API.
La integridad referencial será responsabilidad de la base de datos en v2.

Estado: vigente

D-v1-21 — area_conocimiento como primer CRUD de referencia
Alternativas
Implementar las seis tablas simultáneamente.
Comenzar con la tabla más sencilla.
Utilizar area_conocimiento como CRUD de referencia para establecer el patrón.
Decisión

El primer CRUD completo de la v1 será:

area_conocimiento

Se utilizará para validar el patrón general de desarrollo:

Frontend
    ↓
API Controller
    ↓
Service
    ↓
Repository
    ↓
PDO
    ↓
MariaDB

Una vez validado el patrón, se reutilizará la estructura para las otras cinco tablas.

Consecuencias
Los problemas arquitectónicos podrán detectarse temprano.
Se evitará repetir errores en los otros CRUD.
La estructura creada para area_conocimiento servirá como referencia para el resto de la v1.
Los seis CRUD deberán mantener el mismo contrato general.

Estado: vigente

D-v1-22 — Contrato HTTP como frontera entre frontend y API
Alternativas
Permitir que el frontend conozca detalles internos del backend.
Permitir acceso directo del frontend a la base de datos.
Utilizar el API como frontera de comunicación.
Decisión

El API será la única frontera entre frontend y backend.

El frontend conocerá únicamente:

las URLs de los endpoints;
los métodos HTTP;
los parámetros;
el formato JSON de las solicitudes;
el formato JSON de las respuestas;
los códigos HTTP esperados.

El frontend no conocerá:

las consultas SQL;
las credenciales de MariaDB;
la estructura interna de los Repositories;
la lógica interna de los Services.
Consecuencias
Se reduce el acoplamiento entre frontend y backend.
La API podrá evolucionar internamente sin romper necesariamente el frontend.
Se podrá reutilizar la API para otros clientes.
Los cambios en el contrato HTTP deberán documentarse antes de implementarse.

Estado: vigente

D-v1-23 — Cierre de la versión 1 mediante validación end-to-end
Alternativas
Considerar terminada la v1 cuando el código compile.
Considerar terminada la v1 cuando existan los seis Controllers.
Considerar terminada la v1 cuando los seis CRUD funcionen de extremo a extremo.
Decisión

La versión 1 se considerará terminada únicamente cuando los seis CRUD funcionen de extremo a extremo:

Frontend
    ↓
HTTP
    ↓
API
    ↓
Service
    ↓
Repository
    ↓
MariaDB

Los seis CRUD deberán permitir como mínimo:

listar registros;
consultar registros;
crear registros;
actualizar registros;
eliminar lógicamente registros;
ocultar registros inactivos en las consultas normales;
mostrar errores de forma consistente.

Además, los datos de referencia correspondientes deberán estar cargados y las pruebas manuales deberán haber sido realizadas.

La versión podrá cerrarse cuando se cumplan los criterios definidos para v1 y posteriormente etiquetarse como:

v1
Consecuencias
La existencia de archivos incompletos no será suficiente para cerrar la versión.
Cada CRUD deberá probarse desde el frontend hasta la base de datos.
Los errores detectados durante las pruebas deberán corregirse antes del cierre.
El cierre de v1 servirá como punto estable para comenzar el desarrollo de v2.

Estado: vigente

Resumen de decisiones
ID	Decisión	Estado
D-v1-1	PHP puro, sin framework y sin Composer	Vigente
D-v1-2	MariaDB como motor de base de datos	Vigente
D-v1-3	Seis tablas iniciales para v1	Vigente
D-v1-4	Eliminación lógica mediante activo	Vigente
D-v1-5	Filtrado de registros inactivos	Vigente
D-v1-6	Arquitectura organizada por capas	Vigente
D-v1-7	El Service no conoce HTTP	Vigente
D-v1-8	El Repository es responsable del SQL	Vigente
D-v1-9	PDO y consultas preparadas	Vigente
D-v1-10	Diferencia entre PUT y PATCH	Vigente
D-v1-11	Formato uniforme de errores	Vigente
D-v1-12	Frontend separado del API	Vigente
D-v1-13	Bootstrap para la interfaz	Vigente
D-v1-14	Separación de listados y formularios	Vigente
D-v1-15	termino como PK de termino_clave	Vigente
D-v1-16	Carga de datos de referencia del Excel	Vigente
D-v1-17	Credenciales mediante variables de entorno	Vigente
D-v1-18	Pruebas del Service con repositorios falsos	Vigente
D-v1-19	Autenticación y autorización en v3	Vigente
D-v1-20	Relaciones y claves foráneas en v2	Vigente
D-v1-21	area_conocimiento como primer CRUD	Vigente
D-v1-22	API como frontera entre frontend y backend	Vigente
D-v1-23	Cierre de v1 mediante validación end-to-end	Vigente
Relación con las siguientes versiones

Las decisiones tomadas en v1 deben permitir que el proyecto evolucione sin rehacer completamente la arquitectura.

v2

Se incorporarán:

las relaciones entre entidades;
claves foráneas;
los CRUD restantes;
listas desplegables para campos relacionados;
integridad referencial;
las tablas de relación correspondientes.
v3

Se incorporarán:

usuario;
rol;
rol_usuario;
login;
JWT;
middleware de autenticación;
autorización por roles;
menú según rol;
restricciones de escritura para usuarios de consulta.
v4

Se incorporarán:

consultas que involucren cuatro o más tablas;
dashboard;
gráficos;
páginas corporativas;
imagen propia;
diseño responsive/PWA;
publicación del sistema.

Las decisiones de v1 deberán mantenerse compatibles con esta evolución.

