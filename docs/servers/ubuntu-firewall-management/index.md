---
layout: default
title: Instalación y Configuración de PostgreSQL en Ubuntu
---

# Administración de Firewall en Servidores Ubuntu

La administración de la seguridad de nuestros servidores es tan importante como la calidad de las aplicaciones que se ejecutan en ellos. En esta entrada vamos a ver todo lo que necesitamos para gestionar de forma segura las conexiones hacia nuestro servidor usando un cortafuegos (Firewall).

Altamente versatil, potente y sencillo, hoy veremos como trabajar con **Ubuntu FireWall** (UFW)

#### Contenido

- [Conocer el estado del servicio de firewall](#)
- [Conocer el estado del firewall](#)
- [Habilitar, deshabilitar y reiniciar el firewall](#)
- [Perfiles de aplicaciones](#)
- [Reglas de tráfico](#)
  - [Habilitar un puerto específico](#)
  - [Permitir IP específica hacia un puerto](#)
  - [Permitir IP específica hacia un aplicación](#)
  - [Bloquear una IP](#)
  - [Bloquear una subred](#)
  - [Listar las reglas del firewall](#)
  - [Eliminar una regla del firewall](#)
- [Gestión de logs](#)
  - [Estado del logging](#)
  - [Niveles de logging](#)

Esto ha sido todo por hoy, con los conocimientos expuestos puedes gestionar una gran parte de la seguridad que se requiere para mantener un sistema en línea, expuesto a internet. Pero esto no es suficiente, existen muchas amenzas ahí afuera y también herramientas que nos ayudan a gestionarlas. En la próxima entrada hablaremos de un mecanismo para ayudarnos a bloquear automáticamente amenazas.

---

Autor: Julio César Echeverri M.
