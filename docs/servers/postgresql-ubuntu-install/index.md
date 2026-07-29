---
layout: default
title: Instalación y Configuración de PostgreSQL en Ubuntu
---

## Instalación, Configuración y Uso de PostgreSQL en Ubuntu

Actualizamos el índice de paquetes del sistema operativo y posteriormente instalamos el paquete `postgresql`.

```bash
sudo apt update

sudo apt install postgresql
```

Para confirmar que la instalación fue exitosa, vamos a movernos temporalmente a una sesión con el usuario `postgres` creado en el sistema operativo durante la instalación:

```bash
sudo -u postgres psql
```

Ahora ejecutamos la siguiente consulta para obtener la versión de PostgreSQL instalada.

```sql
SELECT version();
```

## Configuración de PostgreSQL

Por defecto, el motor de PostgreSQL atiende las conexiones a través del puerto `5432`. Sin embargo, puedes obtener información sobre el motor de base de datos con el siguiente comando:

```bash
pg_lsclusters
```

Como resultado de este comando, podrás ver el número de puerto de cada instancia en tu cluster, además de las rutas de datos y de logs.

Otra forma de encontrar el puerto utilizado por la base de datos es a través del clásico comando `ss`:

```bash
sudo ss -tlnp | grep postgres
```

Finalmente para modificar parámetros de funcionamiento de la base de datos (incluyendo el puerto de escucha), podrás acceder a toda la configuración en la ruta que se muestra a continuación, donde debes cambiar `<version>` por la versión que corresponda a tu instalación (16, 17, 18, etc):

```
/etc/postgresql/<version>/main/postgresql.conf
```

## Gestión del Servicio

La base de datos funciona como cualquier otro servicio de Linux, puedes iniciarla, detenerla y verificar su estado actual.


## Interacción con la base de datos en la terminal


## Instalación y conexión con pgAdmin

