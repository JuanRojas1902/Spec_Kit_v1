# Plan técnico — v1: CRUD de las 6 tablas iniciales

## 1. Objetivo de la versión v1

La versión `v1` implementará un sistema web para administrar la información inicial del módulo de **Investigación**.

La solución estará dividida en dos repositorios privados:

- **API:** PHP + POO + MariaDB.
- **Frontend:** PHP + Bootstrap.

En esta primera versión se implementará el CRUD completo de las siguientes seis tablas:

1. `area_conocimiento`
2. `objetivo_desarrollo_sostenible`
3. `area_aplicacion`
4. `termino_clave`
5. `universidad`
6. `linea_investigacion`

La versión se considera terminada cuando las seis entidades puedan ser consultadas, creadas, actualizadas y eliminadas desde el Frontend utilizando la API.

La eliminación será lógica cuando la estructura de la base de datos lo permita. Los registros marcados como inactivos no deberán aparecer en las consultas normales del sistema.

---

## 2. Arquitectura general

El proyecto utilizará una arquitectura por capas para separar responsabilidades.

### API

```text
API/
├── public/
│   └── index.php
│
├── src/
│   ├── Controllers/
│   ├── Services/
│   │   └── Interfaces/
│   ├── Repositories/
│   │   └── Interfaces/
│   ├── Models/
│   ├── Assemblers/
│   ├── Exceptions/
│   └── Configuration/
│
├── tests/
│
├── .env.example
├── .gitignore
└── README.md


Frontend/
├── public/
│   ├── css/
│   ├── js/
│   └── bootstrap/
│
├── views/
│   ├── area_conocimiento/
│   ├── objetivo_desarrollo_sostenible/
│   ├── area_aplicacion/
│   ├── termino_clave/
│   ├── universidad/
│   └── linea_investigacion/
│
├── index.php
├── cliente_api.php
├── .env.example
├── .gitignore
└── README.md
