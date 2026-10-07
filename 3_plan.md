# Plan técnico — v1 (PHP + MariaDB)

## 1. Dos proyectos

```text
api_investigacion/     → la API
front_investigacion/   → el frontend
```

El frontend nunca habla con la base de datos: solo le pide cosas a la API.

## 2. Carpetas de la API

```text
api_investigacion/
├── index.php              ← entrada de todas las peticiones
├── controladores/         ← uno por tabla
├── servicios/             ← reglas de negocio (+ ensamblador.php)
├── repositorios/          ← consultas SQL
├── modelos/               ← una clase por tabla
├── excepciones/           ← errores de negocio
├── configuracion/         ← conexión a MariaDB
├── pruebas/
├── .env
├── .env.example
└── .gitignore
```

## 3. ¿Qué hace cada capa?

| Capa | Hace | NO hace |
|---|---|---|
| `index.php` | Lee el método y la ruta, y llama al controlador | SQL ni reglas |
| Controlador | Lee el JSON, valida la forma y responde con el código HTTP | SQL |
| Servicio | Aplica reglas de negocio | Nada de HTTP |
| Repositorio | Ejecuta SQL con PDO | Reglas de negocio |
| Modelo | Guarda los datos de una entidad | SQL |

Camino de una petición:

```text
HTTP → index.php → Controlador → Servicio → Repositorio → MariaDB
```

La respuesta vuelve por el mismo camino.

## 4. Interfaces y pruebas

Cada servicio y repositorio tiene una **interfaz** (`IServicio...`, `IRepositorio...`). Así, en las pruebas se puede usar un repositorio falso en memoria y no hace falta tener MariaDB encendida.

El archivo `servicios/ensamblador.php` es el único lugar donde se crean los objetos reales y se conectan entre sí.

## 5. Reglas importantes

**Consultas seguras:** siempre con `prepare()` y parámetros. Nunca pegar valores del usuario dentro del SQL.

```php
// Mal
$sql = "SELECT * FROM universidad WHERE id = " . $id;

// Bien
$stmt = $pdo->prepare("SELECT * FROM universidad WHERE id = :id AND activo = 1");
$stmt->execute([':id' => $id]);
```

**Borrado lógico:** `DELETE` no borra la fila, solo cambia `activo` a 0.

```sql
UPDATE universidad SET activo = 0 WHERE id = :id AND activo = 1;
```

**PUT y PATCH:**

- `PUT` exige todos los campos.
- `PATCH` solo valida y cambia los campos que se envían.

## 6. Rutas de la API

Todas las tablas siguen el mismo patrón. Ejemplo:

```http
GET    /api/area_conocimiento
GET    /api/area_conocimiento/{id}
POST   /api/area_conocimiento
PUT    /api/area_conocimiento/{id}
PATCH  /api/area_conocimiento/{id}
DELETE /api/area_conocimiento/{id}
```

Lo mismo para `objetivo_desarrollo_sostenible`, `area_aplicacion`, `termino_clave`, `universidad` y `linea_investigacion`.

## 7. Frontend

```text
front_investigacion/
├── index.php          ← rutas de las pantallas
├── cliente_api.php    ← único archivo que habla con la API
├── vistas/            ← lista y formulario de cada tabla
├── publico/           ← css, js y Bootstrap
└── .env.example
```

El menú tiene: Inicio y las seis tablas.

## 8. Variables de entorno

```env
DB_HOST=
DB_PORT=
DB_DATABASE=investigacion
DB_USERNAME=
DB_PASSWORD=
```

`.env` va en `.gitignore`. Solo se sube `.env.example`.

## 9. Orden de trabajo

1. Crear la base de datos y cargar los datos.
2. Conexión PDO.
3. Modelos e interfaces.
4. Repositorios.
5. Servicios y excepciones.
6. Controladores e `index.php`.
7. Probar la API.
8. Frontend (cliente, vistas y Bootstrap).
9. Probar todo junto.
10. Pull Request, merge y tag `v1`.

## 10. Está listo cuando…

- [ ] Las seis tablas tienen modelo, repositorio, servicio y controlador.
- [ ] Todas las consultas usan `prepare()`.
- [ ] El borrado es lógico y los listados filtran `activo = 1`.
- [ ] El frontend usa solo la API.
- [ ] Hay pruebas con repositorio falso.
- [ ] No hay contraseñas en el código.
