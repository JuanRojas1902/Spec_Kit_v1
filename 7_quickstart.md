# Quickstart — v1: Investigación (PHP + MariaDB)

Los criterios de aceptación de [2_spec.md](2_spec.md) se comprueban mediante
este procedimiento.

La versión 1 se considera terminada cuando los seis CRUD funcionan de extremo
a extremo:

```text
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
```

Las seis tablas de la v1 son:

- `area_conocimiento`
- `objetivo_desarrollo_sostenible`
- `area_aplicacion`
- `termino_clave`
- `universidad`
- `linea_investigacion`

---

## 1. Arrancar el proyecto

Si el proyecto utiliza Docker Compose:

```powershell
docker compose up -d --build
```

Comprobar que los servicios estén activos:

```powershell
docker compose ps
```

La API deberá quedar disponible en:

```text
http://localhost:8000
```

El frontend deberá quedar disponible en el puerto definido por la configuración
del proyecto.

> [!NOTE]
> Si los puertos son diferentes en el entorno local, se deben utilizar los
> puertos definidos en `docker-compose.yml`.

---

## 2. Antes de empezar: pruebas desde PowerShell

En Windows PowerShell 5.1, `curl` es un alias de `Invoke-WebRequest`.

Por esta razón, para evitar problemas con JSON, se utilizará
`Invoke-RestMethod`.

Primero definir la dirección de la API:

```powershell
$API = "http://localhost:8000"
```

Después se puede crear el siguiente ayudante:

```powershell
function pedir($metodo, $ruta, $cuerpo) {
    try {
        if ($null -eq $cuerpo) {
            Invoke-RestMethod `
                -Method $metodo `
                -Uri ($API + $ruta) `
                -ContentType "application/json"
        }
        else {
            Invoke-RestMethod `
                -Method $metodo `
                -Uri ($API + $ruta) `
                -ContentType "application/json" `
                -Body $cuerpo
        }
    }
    catch {
        $respuesta = $_.Exception.Response

        if ($null -ne $respuesta) {
            $lector = New-Object IO.StreamReader(
                $respuesta.GetResponseStream()
            )

            "HTTP " + [int]$respuesta.StatusCode + " -> " +
                $lector.ReadToEnd()
        }
        else {
            $_.Exception.Message
        }
    }
}
```

Este ayudante permite ejecutar los mismos tipos de solicitudes que utilizará
el frontend.

---

## 3. Criterios de aceptación de la API

### 3.1. Diagnóstico de la API

La primera prueba comprueba que la API esté funcionando.

```powershell
pedir GET /
```

**Resultado esperado**

```text
HTTP 200
```

Con una respuesta similar a:

```json
{
    "mensaje": "API de Investigación funcionando",
    "version": "v1",
    "modulo": "Investigación"
}
```

---

## 4. Comprobar que la base de datos inicia con datos

La v1 debe arrancar con los datos de referencia cargados.

Comprobar `area_conocimiento`:

```powershell
(pedir GET /api/area_conocimiento).total
```

**Resultado esperado**

```text
218
```

La tabla debe contener las 218 áreas de conocimiento iniciales.

También se deben comprobar los demás catálogos:

```powershell
(pedir GET /api/objetivo_desarrollo_sostenible).total
```

Resultado esperado: `17`

```powershell
(pedir GET /api/area_aplicacion).total
```

Resultado esperado: `21`

```powershell
(pedir GET /api/universidad).total
```

Resultado esperado: `6`

Los registros iniciales de `termino_clave` y `linea_investigacion` deberán
corresponder a los datos definidos para el proyecto.

---

## 5. CRUD de referencia: `area_conocimiento`

Se utiliza `area_conocimiento` como primer CRUD para comprobar todo el ciclo
de la API.

### 5.1. Crear un registro

Crear un registro temporal para las pruebas:

```powershell
pedir POST /api/area_conocimiento '{
    "id": 9999,
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Prueba de API"
}'
```

**Resultado esperado**

```text
HTTP 200
```

Con una respuesta similar a:

```json
{
    "estado": 200,
    "mensaje": "Área de conocimiento creada exitosamente.",
    "filasAfectadas": 1
}
```

### 5.2. Listar

Después de crear el registro:

```powershell
(pedir GET /api/area_conocimiento).total
```

**Resultado esperado**

```text
219
```

La respuesta deberá incluir el registro creado.

### 5.3. Obtener un registro

```powershell
pedir GET /api/area_conocimiento/9999
```

**Resultado esperado**

```json
{
    "id": 9999,
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Prueba de API"
}
```

---

## 6. Diferencia entre PUT y PATCH

Esta prueba es **obligatoria** porque la API utiliza ambos métodos.

### 6.1. `PUT` — reemplazo completo

Enviar todos los campos editables:

```powershell
pedir PUT /api/area_conocimiento/9999 '{
    "gran_area": "Ingeniería y Tecnología",
    "area": "Ingeniería de Sistemas",
    "disciplina": "Prueba PUT"
}'
```

**Resultado esperado**

```text
HTTP 200
```

Y:

```json
{
    "estado": 200,
    "mensaje": "Área de conocimiento reemplazada.",
    "filasAfectadas": 1
}
```

### 6.2. `PATCH` — actualización parcial

Ahora cambiar solamente la disciplina:

```powershell
pedir PATCH /api/area_conocimiento/9999 '{
    "disciplina": "Prueba PATCH"
}'
```

**Resultado esperado**

```text
HTTP 200
```

Y:

```json
{
    "estado": 200,
    "mensaje": "Área de conocimiento actualizada.",
    "filasAfectadas": 1
}
```

Comprobar:

```powershell
pedir GET /api/area_conocimiento/9999
```

La respuesta debe mostrar:

```text
disciplina = Prueba PATCH
```

### 6.3. El mismo cuerpo debe comportarse diferente

Este cuerpo está incompleto para `PUT`:

```json
{
    "area": "Ingeniería de Sistemas",
    "disciplina": "Prueba"
}
```

Por lo tanto:

```powershell
pedir PUT /api/area_conocimiento/9999 '{
    "area": "Ingeniería de Sistemas",
    "disciplina": "Prueba"
}'
```

**Resultado esperado**

```text
HTTP 422
```

Porque falta:

```text
gran_area
```

Pero el mismo cuerpo sí puede utilizarse mediante `PATCH`:

```powershell
pedir PATCH /api/area_conocimiento/9999 '{
    "area": "Ingeniería de Sistemas",
    "disciplina": "Prueba"
}'
```

**Resultado esperado**

```text
HTTP 200
```

Esto demuestra la diferencia entre reemplazo completo y actualización parcial.

---

## 7. Eliminación lógica

La eliminación de los registros no será física.

Ejecutar:

```powershell
pedir DELETE /api/area_conocimiento/9999
```

**Resultado esperado**

```text
HTTP 200
```

Con:

```json
{
    "estado": 200,
    "mensaje": "Área de conocimiento eliminada.",
    "filasAfectadas": 1
}
```

Volver a listar:

```powershell
(pedir GET /api/area_conocimiento).total
```

**Resultado esperado**

```text
218
```

El registro temporal ya no aparece porque quedó:

```text
activo = 0
```

### 7.1. Segundo `DELETE`

Intentar eliminar nuevamente:

```powershell
pedir DELETE /api/area_conocimiento/9999
```

**Resultado esperado**

```text
HTTP 404
```

La API debe considerar que no existe un registro activo con ese identificador.

### 7.2. Comprobar que la fila sigue en MariaDB

La eliminación debe ser lógica.

Si el proyecto utiliza el servicio `mariadb`, ejecutar:

```powershell
docker compose exec mariadb mariadb `
    -u$env:DB_USER `
    -p$env:DB_PASSWORD `
    investigacion `
    -e "SELECT id, activo FROM area_conocimiento WHERE id = 9999;"
```

El resultado esperado es equivalente a:

```text
9999    0
```

La fila continúa almacenada, pero está inactiva.

> [!WARNING]
> Los valores reales de usuario y contraseña no deben escribirse directamente
> en el repositorio. Deben provenir de las variables de entorno configuradas
> para el proyecto.

---

## 8. Validación de datos

La API debe validar los datos antes de enviarlos al Repository.

Intentar crear un registro incompleto:

```powershell
pedir POST /api/area_conocimiento '{
    "area": "Ingeniería de Sistemas",
    "disciplina": "Ingeniería de software"
}'
```

**Resultado esperado**

```text
HTTP 422
```

La respuesta debe incluir `errores[]`.

Ejemplo:

```json
{
    "estado": 422,
    "mensaje": "Datos inválidos.",
    "errores": [
        "El campo id es obligatorio.",
        "El campo gran_area es obligatorio."
    ]
}
```

La validación ocurre antes de ejecutar el SQL de creación.

---

## 9. Validación de `PATCH` vacío

Enviar un objeto vacío:

```powershell
pedir PATCH /api/area_conocimiento/9999 '{}'
```

**Resultado esperado**

```text
HTTP 400
```

Con una respuesta similar a:

```json
{
    "estado": 400,
    "mensaje": "Parámetros inválidos.",
    "detalle": "No se envió ningún campo para actualizar."
}
```

La diferencia es importante:

```text
422 = la estructura o los datos enviados son inválidos.

400 = la estructura es válida, pero la operación no tiene
      sentido con los datos recibidos.
```

---

## 10. Validación de registro inexistente

Consultar un registro que no existe:

```powershell
pedir GET /api/area_conocimiento/999999
```

**Resultado esperado**

```text
HTTP 404
```

También debe producir `404` un `PUT`, `PATCH` o `DELETE` sobre un registro
inexistente o inactivo.

---

## 11. Prueba de los otros cinco CRUD

Una vez validado completamente `area_conocimiento`, se debe comprobar el
mismo ciclo para las otras cinco tablas.

### 11.1. `objetivo_desarrollo_sostenible`

**Listar**

```powershell
pedir GET /api/objetivo_desarrollo_sostenible
```

Debe devolver:

```text
total = 17
```

**Obtener**

```powershell
pedir GET /api/objetivo_desarrollo_sostenible/1
```

Debe devolver el ODS correspondiente.

**Crear**

Crear un registro temporal:

```powershell
pedir POST /api/objetivo_desarrollo_sostenible '{
    "id": 9999,
    "nombre": "ODS de prueba",
    "categoria": "Social"
}'
```

**Actualizar**

```powershell
pedir PATCH /api/objetivo_desarrollo_sostenible/9999 '{
    "nombre": "ODS actualizado"
}'
```

**Eliminar lógicamente**

```powershell
pedir DELETE /api/objetivo_desarrollo_sostenible/9999
```

Debe responder `200`.

Un segundo `DELETE` debe responder `404`.

### 11.2. `area_aplicacion`

**Listar**

```powershell
pedir GET /api/area_aplicacion
```

Debe devolver:

```text
total = 21
```

**Obtener**

```powershell
pedir GET /api/area_aplicacion/1
```

**Crear**

```powershell
pedir POST /api/area_aplicacion '{
    "id": 9999,
    "nombre": "Área de prueba"
}'
```

**Actualizar**

```powershell
pedir PATCH /api/area_aplicacion/9999 '{
    "nombre": "Área actualizada"
}'
```

**Eliminar**

```powershell
pedir DELETE /api/area_aplicacion/9999
```

Debe responder `200`.

### 11.3. `termino_clave`

Esta tabla debe comprobarse utilizando una clave primaria de texto.

**Listar**

```powershell
pedir GET /api/termino_clave
```

**Crear**

```powershell
pedir POST /api/termino_clave '{
    "termino": "Termino de prueba",
    "termino_ingles": "Test term"
}'
```

**Obtener**

```powershell
pedir GET /api/termino_clave/Termino%20de%20prueba
```

**Actualizar**

```powershell
pedir PATCH /api/termino_clave/Termino%20de%20prueba '{
    "termino_ingles": "Updated test term"
}'
```

**Eliminar**

```powershell
pedir DELETE /api/termino_clave/Termino%20de%20prueba
```

Debe responder `200`.

### 11.4. `universidad`

**Listar**

```powershell
pedir GET /api/universidad
```

Debe devolver:

```text
total = 6
```

**Obtener**

```powershell
pedir GET /api/universidad/1
```

**Crear**

```powershell
pedir POST /api/universidad '{
    "id": 9999,
    "nombre": "Universidad de prueba",
    "tipo": "Pública",
    "ciudad": "Medellín"
}'
```

**Actualizar**

```powershell
pedir PATCH /api/universidad/9999 '{
    "ciudad": "Bogotá"
}'
```

**Eliminar**

```powershell
pedir DELETE /api/universidad/9999
```

Debe responder `200`.

### 11.5. `linea_investigacion`

Esta tabla utiliza `AUTO_INCREMENT`.

Por eso el `POST` **no envía `id`**.

**Listar**

```powershell
pedir GET /api/linea_investigacion
```

**Crear**

```powershell
pedir POST /api/linea_investigacion '{
    "nombre": "Línea de prueba",
    "descripcion": "Descripción de la línea de investigación de prueba."
}'
```

La API debe crear el identificador automáticamente.

**Actualizar**

Utilizar el `id` recibido:

```powershell
pedir PATCH /api/linea_investigacion/{id} '{
    "descripcion": "Descripción actualizada."
}'
```

**Eliminar**

```powershell
pedir DELETE /api/linea_investigacion/{id}
```

Debe responder `200`.

Un segundo `DELETE` debe responder `404`.

---

## 12. Prueba de las capas

La arquitectura de la API debe poder probar la lógica del Service sin
depender directamente de una solicitud HTTP.

Si existe el archivo de pruebas:

```text
pruebas/prueba_capas.php
```

ejecutar:

```powershell
docker compose exec api-investigacion php pruebas/prueba_capas.php
```

**Resultado esperado**

Las verificaciones deberán aparecer como:

```text
[OK]
[OK]
[OK]
...
```

La prueba debe comprobar principalmente:

- validaciones;
- reglas de negocio;
- manejo de registros inexistentes;
- interacción entre Service e interfaz de Repository.

La prueba no debe requerir que el Service conozca HTTP.

---

## 13. Prueba del frontend

Los criterios de API no son suficientes para cerrar la v1.

La versión también necesita una interfaz funcional que consuma el API.

Abrir el frontend en el puerto configurado para el proyecto.

Por ejemplo:

```text
http://localhost:8001
```

El puerto exacto debe coincidir con la configuración del frontend.

### 13.1. `area_conocimiento`

Entrar a:

```text
Áreas de conocimiento
```

Comprobar:

- [ ] Se muestran las 218 filas iniciales.
- [ ] El listado solo muestra registros activos.
- [ ] Existe un botón para agregar.
- [ ] Se puede crear un registro.
- [ ] Aparece un mensaje de éxito.
- [ ] Se puede editar el registro.
- [ ] `PUT` exige todos los campos.
- [ ] `PATCH` permite modificar solo los campos enviados.
- [ ] Se puede retirar el registro.
- [ ] El registro desaparece del listado después del retiro.

### 13.2. Validación desde el formulario

Crear o editar un registro y dejar vacío un campo obligatorio.

Por ejemplo:

```text
gran_area
```

Presionar el botón correspondiente a guardar el registro completo.

**Resultado esperado**

- La API responde `422`.
- El frontend muestra el mensaje de validación.
- Los datos escritos por el usuario no se pierden.
- El registro no se modifica.

### 13.3. Diferencia entre guardar completo y guardar parcialmente

Con el mismo formulario incompleto:

**Guardar completo**

Debe realizar:

```text
PUT
```

y rechazar el formulario incompleto.

**Guardar cambios**

Debe realizar:

```text
PATCH
```

y aceptar únicamente los campos enviados.

Esto demuestra visualmente la diferencia definida en el contrato de la API.

---

## 14. Comprobación de los otros cinco módulos desde el frontend

La pantalla debe permitir acceder a los seis CRUD:

- Áreas de conocimiento
- Objetivos de desarrollo sostenible
- Áreas de aplicación
- Términos clave
- Universidades
- Líneas de investigación

En cada módulo se debe comprobar como mínimo:

- listado;
- creación;
- edición;
- eliminación lógica;
- mensajes de éxito;
- mensajes de error;
- actualización del listado después de una operación.

---

## 15. Prueba de API caída

Esta prueba comprueba que el frontend no se rompa cuando el backend no está
disponible.

Primero detener la API:

```powershell
docker compose stop api-investigacion
```

Abrir o actualizar el frontend.

**Resultado esperado**

La pantalla debe mostrar un mensaje similar a:

```text
El servicio no está disponible.
```

Pero:

- la interfaz no debe quedar en blanco;
- no debe aparecer una excepción de JavaScript sin controlar;
- el usuario debe poder seguir navegando por la interfaz.

Después volver a iniciar la API:

```powershell
docker compose start api-investigacion
```

---

## 16. Problemas frecuentes

| Lo que se ve | Posible causa | Solución |
|---|---|---|
| `Connection refused` en el puerto de la API | La API no está ejecutándose | `docker compose ps` y `docker compose logs api-investigacion` |
| El frontend no puede consultar la API | URL o puerto incorrecto | Revisar la configuración del cliente HTTP |
| El listado devuelve 0 registros | La base se inició sin las semillas | Revisar `db/init.sql` y recrear los volúmenes si es necesario |
| El listado muestra registros eliminados | Falta `WHERE activo = 1` | Revisar el Repository |
| `DELETE` borra físicamente una fila | Se utilizó `DELETE FROM` | Cambiarlo por actualización de `activo` |
| El segundo `DELETE` devuelve `200` | No se está filtrando por `activo = 1` | Revisar el Service y Repository |
| `PUT` acepta campos faltantes | Falta validación completa | Revisar la validación del Controller |
| `PATCH {}` devuelve `200` | No se valida que exista al menos un campo | Revisar el Service |
| `PATCH {}` devuelve `422` | Se está confundiendo validación de forma con regla de negocio | Debe responder `400` |
| La API responde `500` al crear un registro duplicado | Es posible en v1 | La llave duplicada puede reportarse como error interno |
| El `id` de `linea_investigacion` debe enviarse manualmente | Se está ignorando `AUTO_INCREMENT` | El ID debe generarlo MariaDB |
| `termino_clave` no encuentra términos con espacios | Falta codificar el parámetro de URL | Utilizar URL encoding |
| El frontend queda en blanco si cae la API | No se manejó el error de conexión | Mostrar un mensaje de servicio no disponible |
| El frontend no tiene estilos | El servidor/router está interfiriendo con archivos estáticos | Revisar la configuración del servidor PHP |
| Los datos iniciales no aparecen | El volumen de MariaDB ya existía antes de modificar `init.sql` | Recrear el entorno de desarrollo según corresponda |

---

## 17. Comprobación final de la v1

Antes de cerrar la versión, comprobar:

- [ ] La API inicia correctamente.
- [ ] MariaDB inicia correctamente.
- [ ] Las variables de entorno están configuradas.
- [ ] No existen secretos dentro del repositorio.
- [ ] `area_conocimiento` tiene 218 registros iniciales.
- [ ] `objetivo_desarrollo_sostenible` tiene 17 registros iniciales.
- [ ] `area_aplicacion` tiene 21 registros iniciales.
- [ ] `universidad` tiene 6 registros iniciales.
- [ ] `termino_clave` tiene sus datos de referencia.
- [ ] `linea_investigacion` tiene sus datos de referencia.
- [ ] Los seis CRUD tienen `GET` lista.
- [ ] Los seis CRUD tienen `GET` individual.
- [ ] Los seis CRUD tienen `POST`.
- [ ] Los seis CRUD tienen `PUT`.
- [ ] Los seis CRUD tienen `PATCH`.
- [ ] Los seis CRUD tienen `DELETE` lógico.
- [ ] Las consultas normales filtran `activo = 1`.
- [ ] `activo` no puede modificarse desde el cuerpo del frontend.
- [ ] `PUT` exige todos los campos editables.
- [ ] `PATCH` acepta actualización parcial.
- [ ] `PATCH` vacío responde `400`.
- [ ] Registro inexistente responde `404`.
- [ ] Datos inválidos responden `422`.
- [ ] El frontend consume la API correctamente.
- [ ] El frontend muestra los seis módulos.
- [ ] La eliminación lógica funciona desde el frontend.
- [ ] La API caída no rompe la pantalla.
- [ ] Las pruebas de capas pasan.

---

## 18. Cierre de v1

La versión 1 puede cerrarse cuando todos los puntos anteriores estén
comprobados y los seis CRUD funcionen de extremo a extremo.

El flujo completo debe ser:

```text
Usuario
   ↓
Frontend
   ↓
HTTP / JSON
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
PDO
   ↓
MariaDB
```

Una vez comprobado el funcionamiento completo:

1. realizar el commit de los cambios;
2. subir la rama correspondiente;
3. crear el Pull Request;
4. revisar el código;
5. fusionar únicamente mediante el procedimiento definido por el proyecto;
6. etiquetar la versión como:

```text
v1
```

La versión 2 podrá comenzar después de cerrar correctamente esta versión.
