---
layout: default
title: Administración de Firewall en Servidores Ubuntu
---

# Administración de Firewall en Servidores Ubuntu

La administración de la seguridad de nuestros servidores es tan importante como la calidad de las aplicaciones que se ejecutan en ellos. En esta entrada vamos a ver todo lo que necesitamos para gestionar de forma segura las conexiones hacia nuestro servidor usando un cortafuegos (Firewall).

Altamente versatil, potente y sencillo, hoy veremos como trabajar con **Uncomplicated FireWall** (UFW) en Ubuntu.

#### Contenido

- [Instalación del firewall](#instalacion-del-firewall)
- [Gestionar el servicio de firewall](#gestion-de-servicios)
- [Conocer el estado del firewall](#estado-de-firewall)
- [Habilitar, deshabilitar y reiniciar el firewall](#gestion-de-firewall)
- [Perfiles de aplicaciones](#perfiles-aplicacion-firewall)
  - [Información de los perfiles](#informacion-de-perfiles)
  - [Habilitar y deshabilitar perfiles](#habilitar-deshabilitar-perfiles)
  - [Crear y actualizar perfiles](#crear-actualizar-perfiles)
- [Reglas de tráfico](#reglas-de-trafico)
  - [Habilitar conexiones un puerto específico](#habilitar-conexiones-a-puerto)
  - [Permitir conexiones a una aplicación](#conexiones-usando-perfiles)
  - [Permitir conexiones directas a una aplicación](#conexiones-ip-usando-perfiles)
  - [Bloquear una dirección IP](#bloquear-direccion-ip)
  - [Bloquear una subred](#bloquear-subred)
  - [Listar las reglas del firewall](#listar-reglas-firewall)
  - [Eliminar una regla del firewall](#eliminar-reglas-firewall)
- [Gestión de logs](#logging)
  - [Ubicación de los logs](#ubicacion-registros)
  - [Estado del logging](#estados-de-logging)
  - [Niveles de logging](#niveles-de-logging)

---

### Instalación del firewall {#instalacion-del-firewall}

En primer lugar vamos necesitar instalar el paquete de firewall, para ello vamos a ejecutar las siguientes dos líneas.

```bash
sudo apt update
sudo apt install ufw
```

**Nota**: _ufw_ corresponde a las siglas de la aplicación `Uncomplicated FireWall`.

Ahora debemos hacer unos ajustes iniciales, entre ellos vamos a poner unas configuraciones básicas, permitir todo el tráfico saliente y negar todo el trafico entrante (habilitaremos solo lo necesario más adelante).

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

**Importante**: Configura la conexión SSH antes de activar el firewall, de lo contrario la conexión se cerrará y quedarás sin acceso a tu servidor, esto lo hacemos con la siguiente instrucción.

```bash
sudo ufw allow ssh
```

En caso de que (por seguridad) nuestro servidor use un puerto SSH diferente, ejecutamos la siguiente instrucción:

```bash
sudo ufw allow PUERTO/tcp
```

### Gestionar el servicio de firewall {#gestion-de-servicios}

Como en los demás servicios del sistema operativo, en ubuntu podemos gestionar y monitorear el servicio de firewall con el comando `systemctl`.

Para conocer el estado de este servicio podemos ejecutar la línea:

```bash
sudo systemctl status ufw
```

Para iniciar el servicio si se encuentra inactivo:

```bash
sudo systemctl start ufw
```

Para detener el servicio:

```bash
sudo systemctl stop ufw
```

Para habilitar el inicio automático del servicio en el arranque del sistema operativo:

```bash
sudo systemctl enable ufw
```

Para deshabilitar el inicio automático:

```bash
sudo systemctl disable ufw
```

### Conocer el estado del firewall {#estado-de-firewall}

Que el servicio se esté ejecutando no quiere decir que el firewall esté habilitado (que esté bloqueando activamente conexiones), por defecto el firewall no está activo, para evitar que te quedes fuera de tu servidor. Para conocer el estado del firewall podemos ejecutar la siguiente instrucción:

```bash
sudo ufw status [verbose]
```

Cuando se utiliza la opción `verbose` nos mostrará información más detallada, como el nivel de logging actual o las políticas por defecto.

Una opción adicional del comando `status` nos permite listar de forma numerada las reglas activas del firewall, para ellos ejecutamos la siguiente línea:

```bash
sudo ufw status numbered
```

También podemos conocer las reglas agregadas al firewall antes de habilitarlo usando el siguiente comando:

```bash
sudo ufw show added
```

Además, podemos ver los puertos en los cuales se están escuchando conexiones activamente:

```bash
sudo ufw show listening
```

Esto nos basta para conocer el estado general de nuestro firewall, ahora vamos a ver como gestionar el servicio de cara al sistema operativo.

### Habilitar, deshabilitar y reiniciar el firewall {#gestion-de-firewall}

Al igual que con el servicio, el estado del firewall también puede ser gestionado, podemos habilitar, reiniciar o detener las actividades de cortafuegos.

Para activar el cortafuegos y que además se inicie con el sistema operativo, ejecutamos la instrucción:

```bash
sudo ufw enable
```

Para desactivar el cortafuegos y que además no se inicie con el sistema operativo:

```bash
sudo ufw disable
```

Por otra parte, cuando realizamos algunas operaciones, se requiere reiniciar o "recargar" el firewall, para ello podemos ejecutar el siguiente comando:

```bash
sudo ufw reload
```

### Perfiles de aplicaciones {#perfiles-aplicacion-firewall}

En nuestro sistema pueden haber multitud de aplicaciones que aceptan conexiones para brindar un flujo de datos, podemos mejorar la gestión del cortafuegos a través del uso de los llamados _perfiles de aplicación_, que son archivos de texto que describen reglas específicas para un aplicación. Muchas aplicaciones traen sus perfiles por defecto, por ejemplo, los servidores web.

#### Información de los perfiles {#informacion-de-perfiles}

Para listar los perfiles de aplicación disponibles en nuestros sistema podemos usar el comando:

```bash
sudo ufw app list
```

Para obtener información detallada de uno de los perfiles podemos ejecutar:

```bash
sudo ufw app info NombrePerfil
```

Si el nombre del perfil contiene espacios debemos usar comillas, por ejemplo:

```bash
sudo ufw app info "Nombre Perfil"
```

#### Habilitar y deshabilitar perfiles {#habilitar-deshabilitar-perfiles}

Para habilitar (activar) un perfil de aplicación tenemos el siguiente comando:

```bash
sudo ufw allow NombrePerfil
```

Mientras que para deshabilitar un perfil tenemos:

```bash
sudo ufw delete allow NombrePerfil
```

#### Crear y actualizar perfiles {#crear-actualizar-perfiles}

Los archivos que definen los perfiles de las aplicaciones pueden hallarse en el directorio:

```bash
/etc/ufw/applications.d/
```

Estos perfiles son archivos de texto plano con extensión `.profile` y con una estructura como la siguiente:

```bash
[Apache Full]
title=Web Server (HTTP,HTTPS)
description=Apache v2 is the next generation of the omnipresent Apache web server.
ports=80,443/tcp
```

Donde el nombre del perfil es el que aparece entre `[]`.

Si hemos realizado algún cambio en los archivos de perfiles podemos ejecutar el siguiente comando para que el motor de reglas actualize su vigilancia:

```bash
sudo ufw app update NombrePerfil
```

### Reglas de tráfico {#reglas-de-trafico}

En esta sección vamos a estar revisando el paso a paso para la gestión de las reglas de firewall, cómo permitir conexiones a puertos o aplicaciones y cómo realizar bloqueos, tanto a direcciones IP específicas o a subredes enteras.

#### Habilitar conexiones un puerto específico {#habilitar-conexiones-a-puerto}

La notación para habilitar conexiones TCP a un puerto es la siguiente (a modo de ejemplo):

```bash
sudo ufw allow 22/tcp
```

Por otra parte, si no especificamos el protocolo, las conexiones se habilitaran tanto en UDP como TCP.

```bash
# Habilitar conexión en el puerto 22 para UDP y TCP
sudo ufw allow 22
```

#### Permitir conexiones a una aplicación {#conexiones-usando-perfiles}

Como vimos en el apartado de perfiles, para permitir conexiones a aplicaciones descritas en un archivo de perfil, solo necesitamos ejecutar la siguiente instrucción:

```bash
sudo ufw allow NombrePerfil
```

Si el nombre del perfil contiene espacios, usamos comillas dobles:

```bash
sudo ufw allow "nombre del perfil"
```

#### Permitir conexiones directas a una aplicación {#conexiones-ip-usando-perfiles}

Por otra parte, podemos aceptar conexiones hacia una aplicación, que provengan de una dirección IP específica. Para ello necesitamos identificar la dirección IP pública que deseamos configurar, lo podemos hacer con la línea:

```bash
curl -4 icanhazip.com
```

Luego podemos ejecutar la siguiente instrucción para relaciónar un perfil con esa dirección IP:

```bash
sudo ufw allow from <IP> to any app <NombrePerfil> comment "comentario"
```

#### Bloquear una dirección IP {#bloquear-direccion-ip}

Si por alguna razón deseamos bloquear una dirección IP específica, pdemos ejecutar la siguiente instrucción:

```bash
sudo ufw deny from <IP>
```

#### Bloquear una subred {#bloquear-subred}

No tenemos que realizar bloqueos una a una las direcciones IP, podemos bloquear las conexiones de una subred completa:

```bash
sudo ufw deny from <IP/mask>
```

Por ejemplo:

```bash
sudo ufw deny from 203.0.113.100/24
```

#### Listar las reglas del firewall {#listar-reglas-firewall}

Como vimos en la sección de **conocer el estado del firewall**, si deseamos listar las reglas presentes en el firewall, podemos ejecutar la siguiente instrucción:

```bash
sudo ufw status numbered
```

#### Eliminar una regla del firewall {#eliminar-reglas-firewall}

Para aliminar una regla de nuestro firewall, primero debemos conocer su identificador como vimos en la sección anterior y, posteriormente, ejecutar la siguiente línea:

```bash
sudo ufw delete <ID>
```

### Gestión de logs {#logging}

Un aspecto muy importante, casi de cualquier software, es la información de logging, que nos permite analizar en tiempo real o posteriormente las transacciones y los eventos que han ocurrido. En este apartado veremos como se gestionan con **UFW**.

#### Ubicación de los logs {#ubicacion-registros}

Al igual que con la mayoría de aplicaciones que se ejecutan en un servidor Linux, los logs suelen almacenarse en el directorio `/var`. En el caso de `ufw` los logs pueden encontrarse en la ruta:

```bash
   /var/log/ufw.log
```

#### Estado del logging {#estado-de-logging}

Podemos saber si el logging de la aplicación está activo o apagado mediante el comando:

```bash
sudo ufw status verbose
```

Tras la ejecución obtendremos una salida como la siguiente:

```bash
Status: active
Logging: off
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip
...
```

Podemos encender el logging con:

```bash
sudo ufw logging on
```

Podemos detener el logging con:

```bash
sudo ufw logging off
```

#### Niveles de logging {#niveles-de-logging}

Similar a otras herramientas, _UFW_ cuenta con diferentes niveles de logging que nos muestran diferentes niveles de información en los logs. A continuación se listan los 5 niveles desde la documentación oficial:

- **`off`**: Means logging is disabled.
- **`low`**: Will store logs related to blocked packets that do not match the current firewall rules and will show log entries related to logged rules.
- **`medium`**: In addition to all the logs offered by the low level, you get logs for invalid packets, new connections, and logging done through rate limiting.
- **`high`**: Will include logs for packets with rate limiting and without rate limiting.
- **`full`**: This level is similar to the high level but does not include the rate limiting.

En la descripción del nivel `low` puedes ver que se menciona `logged rules`, y es que es posible parametrizar las reglas para que guarden un registro cuando un paquete coincide con la regla, no todas las interesacciones con las reglas generan logs, para activar esta opción manualmente en una regla usamos la opción **log** (registra la primera interacción) o **log-all** (registra todos los paquetes que interactuan con la regla), veamos la sintaxis:

```bash
sudo ufw allow log <puerto o perfil>
```

Si quetemos cambiar el nivel de logging de nuestro firewall ejecutamos la siguiente sintaxis, donde `logging_level` representa uno de los valores listados anteriormente.

```bash
sudo ufw logging <logging_level>
```

Por ejemplo:

```bash
sudo ufw logging medium
```

Verificamos el cambio con:

```bash
sudo ufw status verbose
```

Esto ha sido todo por hoy, con los conocimientos expuestos puedes gestionar una gran parte de la seguridad que se requiere para mantener un sistema expuesto a internet. Pero esto no es suficiente, existen muchas amenzas ahí afuera y también herramientas que nos ayudan a gestionarlas. En la próxima entrada hablaremos de un mecanismo para ayudarnos a bloquear automáticamente ataques de fuerza bruta.

---

Autor: Julio César Echeverri M.
