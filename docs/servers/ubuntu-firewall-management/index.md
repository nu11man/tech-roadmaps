---
layout: default
title: Administración de Firewall en Servidores Ubuntu
---

# Administración de Firewall en Servidores Ubuntu

La administración de la seguridad de nuestros servidores es tan importante como la calidad de las aplicaciones que se ejecutan en ellos. En esta entrada vamos a ver todo lo que necesitamos para gestionar de forma segura las conexiones hacia nuestro servidor usando un cortafuegos (Firewall).

Altamente versatil, potente y sencillo, hoy veremos como trabajar con **Uncomplicated FireWall** (UFW)

#### Contenido

- [Instalación del firewall](#instalacion-del-firewall)
- [Gestionar el servicio de firewall](#gestion-de-servicios)
- [Conocer el estado del firewall](#estado-de-firewall)
- [Habilitar, deshabilitar y reiniciar el firewall](#gestion-de-firewall)
- [Perfiles de aplicaciones](#perfiles-aplicacion-firewall)
- [Reglas de tráfico](#)
  - [Habilitar un puerto específico](#)
  - [Permitir conexiones a un puerto](#)
  - [Permitir conexiones a una aplicación](#)
  - [Bloquear una dirección IP](#)
  - [Bloquear una subred](#)
  - [Listar las reglas del firewall](#)
  - [Eliminar una regla del firewall](#)
- [Gestión de logs](#)
  - [Ubicación de los logs](#)
  - [Estado del logging](#)
  - [Niveles de logging](#)

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

Para habilitar (activar) un perfil de aplicación tenemos el siguiente comando:

```bash
sudo ufw allow NombrePerfil
```

Mientras que para deshabilitar un perfil tenemos:

```bash
sudo ufw delete allow NombrePerfil
```

### Reglas de tráfico {#}

#### Habilitar un puerto específico {#}

#### Permitir conexiones a un puerto {#}

#### Permitir conexiones a una aplicación {#}

#### Bloquear una dirección IP {#}

#### Bloquear una subred {#}

#### Listar las reglas del firewall {#}

#### Eliminar una regla del firewall {#}

### Gestión de logs {#}

#### Ubicación de los logs {#}

#### Estado del logging {#}

#### Niveles de logging {#}

Esto ha sido todo por hoy, con los conocimientos expuestos puedes gestionar una gran parte de la seguridad que se requiere para mantener un sistema expuesto a internet. Pero esto no es suficiente, existen muchas amenzas ahí afuera y también herramientas que nos ayudan a gestionarlas. En la próxima entrada hablaremos de un mecanismo para ayudarnos a bloquear automáticamente ataques de fuerza bruta.

---

Autor: Julio César Echeverri M.
