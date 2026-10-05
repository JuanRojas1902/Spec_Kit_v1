# Plan técnico — v1: Catálogos de Investigación (PHP + MariaDB)

## 1. Estructura del proyecto

El proyecto estará dividido en dos repositorios privados:

```text
api_investigacion/
└── API

front_investigacion/
└── Frontend

La API será responsable de:

Recibir peticiones HTTP.
Validar la estructura de los datos recibidos.
Aplicar las reglas de negocio.
Ejecutar las consultas SQL.
Comunicarse con MariaDB.
Devolver respuestas JSON.

El Frontend será responsable de:

Mostrar las interfaces al usuario.
Mostrar formularios.
Validar información básica antes de enviarla.
Consumir la API.
Mostrar resultados, errores y mensajes de operación.
Utilizar Bootstrap para la interfaz.
2. Árbol de la API

La estructura propuesta para la API es:

api_investigacion/
├── index.php
│
├── controladores/
│   ├── ControladorAreaConocimiento.php
│   ├── ControladorObjetivoDesarrolloSostenible.php
│   ├── ControladorAreaAplicacion.php
│   ├── ControladorTerminoClave.php
│   ├── ControladorUniversidad.php
│   └── ControladorLineaInvestigacion.php
│
├── servicios/
│   ├── IServicioAreaConocimiento.php
│   ├── ServicioAreaConocimiento.php
│   ├── IServicioObjetivoDesarrolloSostenible.php
│   ├── ServicioObjetivoDesarrolloSostenible.php
│   ├── IServicioAreaAplicacion.php
│   ├── ServicioAreaAplicacion.php
│   ├── IServicioTerminoClave.php
│   ├── ServicioTerminoClave.php
│   ├── IServicioUniversidad.php
│   ├── ServicioUniversidad.php
│   ├── IServicioLineaInvestigacion.php
│   ├── ServicioLineaInvestigacion.php
│   └── ensamblador.php
│
├── repositorios/
│   ├── IRepositorioAreaConocimiento.php
│   ├── RepositorioAreaConocimientoMariaDB.php
│   ├── IRepositorioObjetivoDesarrolloSostenible.php
│   ├── RepositorioObjetivoDesarrolloSostenibleMariaDB.php
│   ├── IRepositorioAreaAplicacion.php
│   ├── RepositorioAreaAplicacionMariaDB.php
│   ├── IRepositorioTerminoClave.php
│   ├── RepositorioTerminoClaveMariaDB.php
│   ├── IRepositorioUniversidad.php
│   ├── RepositorioUniversidadMariaDB.php
│   ├── IRepositorioLineaInvestigacion.php
│   └── RepositorioLineaInvestigacionMariaDB.php
│
├── modelos/
│   ├── AreaConocimiento.php
│   ├── ObjetivoDesarrolloSostenible.php
│   ├── AreaAplicacion.php
│   ├── TerminoClave.php
│   ├── Universidad.php
│   └── LineaInvestigacion.php
│
├── excepciones/
│   ├── NoEncontradoExcepcion.php
│   ├── ConflictoExcepcion.php
│   └── ValidacionExcepcion.php
│
├── configuracion/
│   └── ConexionMariaDB.php
│
├── pruebas/
│   ├── prueba_area_conocimiento.php
│   ├── prueba_objetivo_desarrollo_sostenible.php
│   ├── prueba_area_aplicacion.php
│   ├── prueba_termino_clave.php
│   ├── prueba_universidad.php
│   └── prueba_linea_investigacion.php
│
├── .env
├── .env.example
├── .gitignore
└── README.md
