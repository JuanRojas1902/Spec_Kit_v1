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

# 2. Fuera de alcance

En la versión v1 NO se implementarán:

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

# 3. Tablas incluidas en v1

## 3.1 area_conocimiento

Tabla utilizada para almacenar las áreas del conocimiento.

### Campos

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `id` | INT | Sí | Identificador del área de conocimiento |
| `gran_area` | VARCHAR(60) | Sí | Gran área de conocimiento |
| `area` | VARCHAR(60) | Sí | Área de conocimiento |
| `disciplina` | VARCHAR(60) | Sí | Disciplina asociada |

### Llave primaria

```text
id
