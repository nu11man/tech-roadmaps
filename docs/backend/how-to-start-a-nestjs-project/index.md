---
layout: default
title: Iniciar un Projecto con NestJS
---

# Iniciar un Proyecto con NestJS

Los inicios suelen ser de vital importancia en casi cualquier aspecto de la vida, así mismo con nuestros proyectos. En este artículo veremos como iniciar un proyecto de NestJS y sus configuraciones iniciales.

## Instalación de NestJS CLI

En primer lugar debemos instalar la CLI (Command Line Interface) de NestJS que entre otras cosas nos ayuda a crear proyectos o sus componentes internos.

```bash
npm install --global @nestjs/cli
```

Con el asistente de NestJS instalado, podemos crear un proyecto completamente nuevo o crearlo dentro de un directorio definido previamente y además indicarle que no sobreescriba configuraciones de `git`. Ambas opciones las podemos ver a continuación:

```bash
# Crea una nueva carpeta project_name dentro de la ruta actual
nest new project_name


# Crea el proyecto en el directorio actual y no sobreescribe configuraciones previas de git
nest new . --skip-git
```

Cuando ejecutamos el comando de creación, el asistente nos guiará a través de algunas preguntas sobre la configuración inicial como, por ejemplo, el gestor de depedencias que queremos utilizar (yarn, npm, etc).

Una vez finalizado el proceso de creación podemos ejecutar las siguientes acciones:

- `yarn run start`: Inicia el servidor y comienza a escuchar peticiones por el puerto definido.
- `yarn run start:dev`: Inicia el servidor en modo development (escucha cambios en los archivos y reinicia)
- `yarn run format`: Ejecuta el plugin de `prettier` para dar formato al código fuente.
- `yarn run lint`: Ejecuta el linter `ESLint` y corrige o muestra los errores propios de esta herramienta.

## Configuración de Alias en los Imports

