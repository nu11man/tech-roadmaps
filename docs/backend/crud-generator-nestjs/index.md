---
layout: default
title: Resource CRUD Generator de NestJS
---

# Resource CRUD Generator de NestJS

Cuando estamos desarrollando REST APIs es muy frecuente crear módulos con las operaciones básicas de CRUD (Create, Read, Update y Delete), y que a su vez esto implica generar los endpoints, el service, los DTO y las entities de base de datos. NestJS nos ofrece un esquemático que al ejecutarlo crea el andamiaje (también llamado _boiler plate_ o _scaffolding_).

Vamos al punto, para generar un nuevo recurso de forma automática siguiendo buenas prácticas, vamos al directorio raíz de nuestro proyecto y ejecutamos el comando:

```bash
nest g|generate resource nombre_recurso
```

Por ejemplo, para generar el recurso `posts` de nuestro blog, ejecutariamos el comando:

```bash
nest generate resource posts
```

Al terminar la ejecución veremos que automáticamente:

- Se creo la carpeta `posts` dentro del directorio `src`.
- Se creó el controlador asociado al recurso.
- Se creó el módulo que soportará este recurso.
- Se creó el servicio que almacenará la lógica de negocio asociada a este recurso.
- Se crearon los DTOs.
- Se creó el directorio de las entidades de base de datos.
- Se actualizó el archivo `app.module.ts` para agregar el nuevo módulo de `posts`.
