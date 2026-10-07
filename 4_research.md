# Decisiones — v1

Aquí se anotan las decisiones que tomamos y por qué.

| # | Decisión | Por qué |
|---|---|---|
| 1 | **PHP puro**, sin framework ni Composer | Es más simple para aprender y tenemos más control |
| 2 | **MariaDB** con la base `investigacion` | Es el motor definido para el proyecto |
| 3 | **Seis tablas** en v1, sin relaciones | Empezar con lo sencillo; las relaciones van en v2 |
| 4 | **Borrado lógico** con el campo `activo` | No se pierden datos |
| 5 | Filtrar `activo = 1` **en el backend** | No depender solo del frontend |
| 6 | **Arquitectura por capas** | Cada parte tiene una sola función |
| 7 | El servicio **no conoce HTTP** | Se puede probar sin servidor web |
| 8 | El **repositorio** es el único que escribe SQL | Las consultas quedan en un solo lugar |
| 9 | **PDO** con consultas preparadas | Evita la inyección SQL |
| 10 | `PUT` reemplaza todo, `PATCH` cambia parte | Cada método con su propósito |
| 11 | Errores siempre en **JSON con el mismo formato** | El frontend los maneja igual siempre |
| 12 | Frontend **separado** de la API | El frontend no ve las credenciales de la base de datos |
| 13 | **Bootstrap** para la interfaz | Ahorra tiempo y es responsive |
| 14 | Pantallas separadas: **lista** y **formulario** | Más fácil de usar y de mantener |
| 15 | `termino_clave` mantiene **`termino` como llave primaria** | Así viene en el SQL oficial |
| 16 | Cargar los **datos de referencia** antes de cerrar v1 | Probar con datos reales |
| 17 | **Variables de entorno** para las credenciales | No subir secretos a Git |
| 18 | Probar los servicios con **repositorios falsos** | Pruebas rápidas sin base de datos |
| 19 | Login y JWT **en v3** | v1 se concentra en los CRUD |
| 20 | Relaciones y llaves foráneas **en v2** | Se hacen después de tener los CRUD |
| 21 | `area_conocimiento` es el **primer CRUD** (el modelo a copiar) | Se detectan problemas temprano |
| 22 | La **API es la frontera** entre frontend y backend | Menos acoplamiento |
| 23 | v1 se cierra cuando todo funciona **de extremo a extremo** | No basta con tener archivos |

## Detalles de algunas decisiones

### Borrado lógico (decisión 4)

Se agrega este campo a las seis tablas:

```sql
activo TINYINT(1) NOT NULL DEFAULT 1
```

`DELETE` cambia `activo` de 1 a 0. Es un cambio respecto al SQL oficial, que no traía este campo, y está documentado en `5_data_model.md`.

### Formato de errores (decisión 11)

```json
{
    "estado": 422,
    "mensaje": "Datos inválidos.",
    "errores": ["El campo nombre es obligatorio."]
}
```

El detalle completo está en `6_api_contract.md`.

### Llave de texto (decisión 15)

```sql
CREATE TABLE termino_clave (
    termino VARCHAR(30) NOT NULL,
    termino_ingles VARCHAR(30),
    PRIMARY KEY (termino)
);
```

Como la llave es texto, en la URL hay que codificar los espacios: `Inteligencia%20artificial`.

### Credenciales (decisión 17)

```env
DB_HOST=
DB_PORT=
DB_DATABASE=investigacion
DB_USERNAME=
DB_PASSWORD=
```

`.env` no se sube a Git. `JWT_SECRET` se agrega hasta la v3.

## Qué viene después

- **v2:** las 10 tablas restantes, llaves foráneas y listas desplegables.
- **v3:** usuarios, roles, login y JWT.
- **v4:** consultas con varias tablas, dashboard, PWA y publicación.
