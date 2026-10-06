# Investigación y decisiones — v1: módulo de Investigación (PHP + MariaDB)

Cada decisión debe indicar las alternativas consideradas, la decisión tomada y
sus consecuencias. Una decisión sin alternativa no es una decisión: es
solamente la primera opción que se pensó.

---

## D-v1-1 — PHP puro, sin framework y sin Composer

**Contexto.** La versión `v1` debe implementar una API en PHP utilizando
Programación Orientada a Objetos y una arquitectura por capas.

**Alternativas.**

(a) Utilizar un framework como Laravel o Slim.

(b) Utilizar PHP puro con las extensiones disponibles, principalmente PDO.

**Decisión: (b).**

Se utilizará PHP puro, sin framework y sin Composer como dependencia
obligatoria del proyecto.

La razón principal es que el objetivo académico del proyecto es comprender
qué ocurre detrás de las herramientas que normalmente resuelve un framework:

- Enrutamiento.
- Lectura de `php://input`.
- Manejo de métodos HTTP.
- Validación.
- Códigos de respuesta HTTP.
- Conversión de datos a JSON.
- Manejo de excepciones.
- Acceso a MariaDB mediante PDO.
- Separación entre Controller, Service y Repository.

**Consecuencias.**

Habrá más código escrito manualmente y algunas responsabilidades que un
framework normalmente resolvería deberán implementarse explícitamente.

A cambio, la arquitectura será visible en el repositorio y se podrá demostrar
el funcionamiento de cada capa.

**Estado:** vigente.

---

## D-v1-2 — MariaDB como motor de base de datos

**Contexto.** El proyecto debe trabajar con la base de datos de Investigación
entregada para el curso.

**Alternativas.**

(a) Utilizar PostgreSQL.

(b) Utilizar MariaDB.

**Decisión: (b).**

Se utilizará MariaDB.

El script SQL entregado para el proyecto utiliza sintaxis compatible con
MySQL/MariaDB y la estructura de la base de datos corresponde al modelo
relacional definido para el módulo de Investigación.

Además, MariaDB permite trabajar directamente con PHP mediante PDO y mantiene
el proyecto alineado con el material utilizado durante el curso.

**Consecuencias.**

Algunas decisiones de implementación estarán relacionadas específicamente con
MariaDB/MySQL, por ejemplo:

- `AUTO_INCREMENT`.
- `TINYINT`.
- Comportamiento de `rowCount()`.
- `MYSQL_ATTR_FOUND_ROWS`.
- Uso de PDO con el driver `mysql`.

Estas decisiones no deberán confundirse con características propias del
lenguaje PHP.

**Estado:** vigente.

---

## D-v1-3 — La v1 se construye con las seis tablas iniciales

**Contexto.** La versión `v1` del proyecto requiere implementar el CRUD de seis
tablas antes de avanzar hacia las relaciones de `v2`.

Las tablas son:

1. `area_conocimiento`
2. `objetivo_desarrollo_sostenible`
3. `area_aplicacion`
4. `termino_clave`
5. `universidad`
6. `linea_investigacion`

**Alternativas.**

(a) Implementar solamente una tabla y dejar las otras para versiones
posteriores.

(b) Implementar las seis tablas definidas para la `v1`.

**Decisión: (b).**

La `v1` debe demostrar que la arquitectura funciona para diferentes tipos de
entidades y no solamente para una tabla.

Las seis tablas también permiten comprobar diferentes situaciones:

- Claves primarias enteras.
- Una clave primaria de tipo texto.
- Campos obligatorios.
- Campos opcionales.
- Campos con diferentes longitudes.
- Un campo `AUTO_INCREMENT`.
- Catálogos.
- Diferentes formularios en el Frontend.

La tabla `area_conocimiento` será utilizada como primer CRUD de referencia
para validar la arquitectura antes de replicarla en los otros cinco módulos.

**Consecuencias.**

La arquitectura deberá ser suficientemente general para no quedar acoplada
a una sola tabla.

Se deberán crear Controllers, Services, Repositories e Interfaces para cada
entidad.

**Estado:** vigente.

---

## D-v1-4 — La eliminación será lógica

**Contexto.**

El requisito funcional de la aplicación establece que los registros
eliminados no deben aparecer en las consultas normales.

Sin embargo, el esquema inicial entregado para las tablas de la `v1` no
contiene un campo `activo`.

**Alternativas.**

(a) Realizar borrado físico utilizando `DELETE FROM`.

(b) Incorporar un campo `activo` y realizar eliminación lógica.

**Decisión: (b).**

Se utilizará eliminación lógica.

La información no será eliminada físicamente de la base de datos. En su lugar,
el registro será marcado como inactivo.

El patrón esperado será:

```sql
activo TINYINT(1) NOT NULL DEFAULT 1


WHERE activo = 1



HTTP
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
MariaDB



No deberá utilizar directamente:

$_GET
$_POST
$_SERVER
http_response_code()
header()
php://input

Estas responsabilidades pertenecen a la capa HTTP/Controller.



interface UniversidadRepositoryInterface
{
    public function listar(): array;

    public function obtenerPorId(int $id): ?array;

    public function crear(array $datos): array;

    public function actualizar(int $id, array $datos): array;

    public function eliminar(int $id): bool;
}

UPDATE tabla
SET activo = 0


SELECT *
FROM universidad
WHERE activo = 1
ORDER BY nombre
WHERE id = :id



Ejemplo:

$sql = "SELECT *
        FROM universidad
        WHERE id = :id
        AND activo = 1";

$stmt = $pdo->prepare($sql);

$stmt->execute([
    ':id' => $id
]);

No se permitirá construir consultas de esta forma:

$sql = "SELECT *
        FROM universidad
        WHERE id = " . $_GET['id'];
