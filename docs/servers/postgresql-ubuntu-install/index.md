---
layout: default
title: Instalación y Configuración de PostgreSQL en Ubuntu
---

## Instalación, Configuración y Uso de PostgreSQL en Ubuntu

Uno de los aspectos esenciales en la gestión de aplicaciones es la capa de persistencia de datos, es decir, el mecanismo de almacenamiento y control de la información. Entre algunos mecanismos, el más destacado por escalabilidad, eficiencia y confiabilidad son los motores bases de datos.

En esta oportunidad veremos como instalar, configurar y poner a punto de operación el motor de base de datos **PostgreSQL**.

#### Contenido

- [Instalación de PostgreSQL]()
- [Configuración de PostgreSQL]()
- [Gestión del Servicio de Base de Datos]()
- [Instalación de pgAdmin]()
- [Gestión de Usuarios desde la Terminal]()

---

### Instalación de PostgreSQL

En primer lugar debemos actualizar el índice de paquetes del sistema operativo.

```bash
sudo apt update
```

Posteriormente instalamos el paquete `postgresql`.

```bash
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

### Configuración de PostgreSQL

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

### Gestión del Servicio de Base de Datos

La base de datos funciona como cualquier otro servicio de Linux, puedes iniciarla, detenerla y verificar su estado actual.

Para gestionar los servicios usamos el comando `systemctl` que tiene la siguiente sintaxis:

```bash
sudo systemctl <action> <service>
```
Donde `service` corresponde al nombre del servicio con el que estamos trabajando (en este caso PostgreSQL) y `action` corresponde a la acción que queremos ejecutar sobre el servicio, esto es, iniciarlo, detenerlo, monitorearlo.

Para conocer el estado actual del servicio ejecutamos la línea:

```bash
sudo systemctl status postgresql
```

Para iniciar el servicio de la base de datos ejecutamos la línea:

```bash
sudo systemctl start postgresql
```

Para detener el servicio de la base de datos ejecutamos la línea:

```bash
sudo systemctl stop postgresql
```

### Instalación y conexión con pgAdmin


### Gestión de Usuarios desde la Terminal

#### Creación de Usuarios y Asignación de Contraseñas

