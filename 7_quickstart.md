# — v1

Con esta guía se comprueba que la v1 funciona de punta a punta.

## 1. Arrancar el proyecto

```powershell
docker compose up -d --build
docker compose ps
```

- API: `http://localhost:8000`
- Frontend: el puerto que diga `docker-compose.yml` (por ejemplo `8001`).

## 2. Preparar PowerShell

En PowerShell, `curl` no sirve bien para JSON. Usamos `Invoke-RestMethod`.

```powershell
$API = "http://localhost:8000"

function pedir($metodo, $ruta, $cuerpo) {
    try {
        if ($null -eq $cuerpo) {
            Invoke-RestMethod -Method $metodo -Uri ($API + $ruta) -ContentType "application/json"
        }
        else {
            Invoke-RestMethod -Method $metodo -Uri ($API + $ruta) -ContentType "application/json" -Body $cuerpo
        }
    }
    catch {
        $r = $_.Exception.Response
        if ($null -ne $r) {
            $lector = New-Object IO.StreamReader($r.GetResponseStream())
            "HTTP " + [int]$r.StatusCode + " -> " + $lector.ReadToEnd()
        }
        else { $_.Exception.Message }
    }
}
```

## 3. La API funciona

```powershell
pedir GET /
```

Debe responder `200` con el mensaje "API de Investigación funcionando".

## 4. Los datos iniciales están cargados

```powershell
(pedir GET /api/area_conocimiento).total              # 218
(pedir GET /api/objetivo_desarrollo_sostenible).total # 17
(pedir GET /api/area_aplicacion).total                # 21
(pedir GET /api/universidad).total                    # 6
```

## 5. Probar `area_conocimiento`

**Crear:**

```powershell
pedir POST /api/area_conocimiento '{"id": 9999, "gran_area": "Ingeniería y Tecnología", "area": "Ingeniería de Sistemas", "disciplina": "Prueba de API"}'
```

Esperado: `200`. Ahora el total debe ser **219**.

**Ver uno:**

```powershell
pedir GET /api/area_conocimiento/9999
```

## 6. Diferencia entre PUT y PATCH

**PUT** con todos los campos → `200`:

```powershell
pedir PUT /api/area_conocimiento/9999 '{"gran_area": "Ingeniería y Tecnología", "area": "Ingeniería de Sistemas", "disciplina": "Prueba PUT"}'
```

**PATCH** con un solo campo → `200`:

```powershell
pedir PATCH /api/area_conocimiento/9999 '{"disciplina": "Prueba PATCH"}'
```

**El mismo cuerpo incompleto** (falta `gran_area`):

```powershell
# PUT → 422 (falta un campo)
pedir PUT /api/area_conocimiento/9999 '{"area": "Ingeniería de Sistemas", "disciplina": "Prueba"}'

# PATCH → 200 (solo cambia lo que se envía)
pedir PATCH /api/area_conocimiento/9999 '{"area": "Ingeniería de Sistemas", "disciplina": "Prueba"}'
```

## 7. Borrado lógico

```powershell
pedir DELETE /api/area_conocimiento/9999   # 200
(pedir GET /api/area_conocimiento).total   # vuelve a 218
pedir DELETE /api/area_conocimiento/9999   # 404 (ya está retirado)
```

Comprobar que la fila **sigue en la base de datos**:

```powershell
docker compose exec mariadb mariadb -u$env:DB_USER -p$env:DB_PASSWORD investigacion -e "SELECT id, activo FROM area_conocimiento WHERE id = 9999;"
```

Debe mostrar `9999  0`.

> [!WARNING]
> Nunca escribas usuario ni contraseña reales en el repositorio. Salen de las variables de entorno.

## 8. Errores esperados

```powershell
# POST incompleto → 422 con lista "errores"
pedir POST /api/area_conocimiento '{"area": "Ingeniería de Sistemas"}'

# PATCH vacío → 400
pedir PATCH /api/area_conocimiento/9999 '{}'

# Registro que no existe → 404
pedir GET /api/area_conocimiento/999999
```

Diferencia: `422` = los datos están mal; `400` = los datos están bien pero la operación no tiene sentido.

## 9. Probar las otras cinco tablas

Repite el ciclo (crear, `PATCH`, `DELETE` y un segundo `DELETE` → `404`) para cada una:

```powershell
# objetivo_desarrollo_sostenible
pedir POST /api/objetivo_desarrollo_sostenible '{"id": 9999, "nombre": "ODS de prueba", "categoria": "Social"}'
pedir PATCH /api/objetivo_desarrollo_sostenible/9999 '{"nombre": "ODS actualizado"}'
pedir DELETE /api/objetivo_desarrollo_sostenible/9999

# area_aplicacion
pedir POST /api/area_aplicacion '{"id": 9999, "nombre": "Área de prueba"}'
pedir PATCH /api/area_aplicacion/9999 '{"nombre": "Área actualizada"}'
pedir DELETE /api/area_aplicacion/9999

# termino_clave (llave de texto, con %20 en la URL)
pedir POST /api/termino_clave '{"termino": "Termino de prueba", "termino_ingles": "Test term"}'
pedir PATCH /api/termino_clave/Termino%20de%20prueba '{"termino_ingles": "Updated term"}'
pedir DELETE /api/termino_clave/Termino%20de%20prueba

# universidad
pedir POST /api/universidad '{"id": 9999, "nombre": "Universidad de prueba", "tipo": "Pública", "ciudad": "Medellín"}'
pedir PATCH /api/universidad/9999 '{"ciudad": "Bogotá"}'
pedir DELETE /api/universidad/9999

# linea_investigacion (sin id: lo genera MariaDB)
pedir POST /api/linea_investigacion '{"nombre": "Línea de prueba", "descripcion": "Descripción de prueba."}'
pedir PATCH /api/linea_investigacion/{id} '{"descripcion": "Descripción actualizada."}'
pedir DELETE /api/linea_investigacion/{id}
```

Para `linea_investigacion`, cambia `{id}` por el id que se creó (míralo en la lista).

## 10. Prueba de capas (sin base de datos)

```powershell
docker compose exec api-investigacion php pruebas/prueba_capas.php
```

Deben salir líneas `[OK]`.

## 11. Probar el frontend

Abre `http://localhost:8001` y comprueba en cada una de las seis pantallas:

- [ ] Se ve la lista con sus datos.
- [ ] Se puede crear un registro y sale mensaje de éxito.
- [ ] Se puede editar.
- [ ] Se puede retirar y desaparece de la lista.
- [ ] Si dejas un campo obligatorio vacío, sale el error y **no se pierde lo que escribiste**.

## 12. Probar con la API apagada

```powershell
docker compose stop api-investigacion
```

Recarga el frontend. Debe decir algo como "El servicio no está disponible", sin quedar en blanco. Luego:

```powershell
docker compose start api-investigacion
```

## 13. Problemas frecuentes

| Qué pasa | Qué revisar |
|---|---|
| `Connection refused` | `docker compose ps` y `docker compose logs api-investigacion` |
| La lista sale vacía | Revisar `db/init.sql` y recrear los volúmenes de la base de datos |
| La lista muestra registros retirados | Falta `WHERE activo = 1` en el repositorio |
| `DELETE` borra de verdad | Debe ser un `UPDATE`, no un `DELETE FROM` |
| El segundo `DELETE` devuelve `200` | Falta `AND activo = 1` |
| `PUT` acepta campos faltantes | Revisar la validación del controlador |
| `PATCH {}` devuelve `200` o `422` | Debe devolver `400` |
| `termino_clave` no encuentra términos con espacios | Falta codificar la URL (`%20`) |
| El frontend no tiene estilos | El router de PHP está bloqueando los archivos estáticos |

## 14. Comprobación final

- [ ] La API y MariaDB arrancan.
- [ ] Los datos iniciales están cargados (218, 17, 21 y 6).
- [ ] Las seis tablas tienen `GET`, `POST`, `PUT`, `PATCH` y `DELETE`.
- [ ] `PATCH` vacío → 400, no existe → 404, datos inválidos → 422.
- [ ] El frontend muestra las seis tablas y retira registros.
- [ ] Si la API se apaga, el frontend no se rompe.
- [ ] La prueba de capas pasa.
- [ ] No hay contraseñas en el repositorio.

## 15. Cerrar v1

1. Hacer commit.
2. Subir la rama.
3. Crear el Pull Request.
4. Revisar y hacer merge a `main`.
5. Crear el tag `v1`.

La v2 empieza solo después de cerrar la v1.
