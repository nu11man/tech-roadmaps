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

### Instalación y conexión de pgAdmin

Un cliente de escritorio o web es de vital importancia para gestionar una base de datos. En este caso nuestra recomendación en pgAdmin ya que es una herramienta de los mismos desarrolladores de PostgreSQL, aunque existen alternativas.

En primer lugar vamos a configurar la llave pública del repositorio de pgAdmin.

```bash
curl -fsS https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo gpg --dearmor -o /etc/apt/keyrings/packages-pgadmin-org.gpg
```

Con la llave agregada vamos ahora a crear el archivo de configuración del repositorio:

```bash
sudo sh -c 'echo "deb [signed-by=/etc/apt/keyrings/packages-pgadmin-org.gpg] https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'
```

Ahora vamos a actualizar el índice de repositorios e instalamos nuestro cliente.

```bash
sudo apt update

sudo apt install pgadmin4
```

El comando anterior instala el cliente web y el cliente de escritorio, puedes instalarlos de forma independiente con los comandos:

```bash
# Install for desktop mode only:
sudo apt install pgadmin4-desktop

# Install for web mode only: 
sudo apt install pgadmin4-web 
```

Si instalas la versión web y deseas usarla debes configurar el servidor web con el siguiente comando:

```bash
# Configure the webserver, if you installed pgadmin4-web:
sudo /usr/pgadmin4/bin/setup-web.sh
```


### Gestión de Usuarios desde la Terminal

Para realizar la gestión más básica de usuarios podemos valernos de la terminal de comandos, esto requiere iniciar la sesión del usuario `postgres` creado automáticamente tras la instalación y ejecutar la herramienta `psql`.

```bash
sudo -u postgres psql
```

#### Actualiza la contraseña de un usuario

Una vez nos encontramos en la shell de Postgres vamos ejecutar el comando `\password` seguido del nombre del usuario al que queremos actualizar o agregar una nueva contraseña, este caso el usuario administrador `postgres`.

```bash
\password postgres
```

Allí asignaremos la nueva contraseña. Este es el método recomendado. Anteriormente utilizabamos una instrucción de SQL para alterar la contraseña del usuario pero esto nos expone al riesgo de filtrar contraseñas en el historial de la terminal o en archivos de logs.

El método antiguo consistía en ejecutar la siguiente instrucción en la shell `psql`:

```sql
ALTER ROLE postgres WITH PASSWORD 'your_new_password';
```

Para salir de la shell `psql` y regresar a nuestro usuario normal del sistema operativo usamos el comando `\q`.

#### Creación de nuevos usuarios

Una de las mayores recomendaciones es mantener el mínimo nivel de privilegios posible, esto es muy importante en cualquier escenario y con postgreSQL podemos lograrlo creando usuarios y gestionando permisos/roles.

Creamos un nuevo usuario y asignamos una contraseña con la siguiente instrucción en `psql`.

```bash
CREATE USER appuser WITH PASSWORD 'my_secure_password';
```

