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

Ya hemos creado y puesto en marcha nuestro servicio. Ahora, vamos a realizar un par de ajustes para mejorar la experiencia de desarrollo, específicamente, en lo relativo a la gestión de los `imports` del código fuente. En otras palabras, pasar de esto:

```typescript
import { utility } from '../../../../utils/utility.ts'
```

a esto:

```typescript
import { utility } from '@utils/utility.ts'
```

Para lograr esto vamos a ajustar, en primer lugar, el archivo `tsconfig.json` y le indicaremos a Typescript cómo realizar el mapeo de las rutas, para esto agregamos el fragmento `paths` en el json:

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@src/*": ["src/*"],
      "@common/*": ["src/common/*"],
      "@modules/*": ["src/modules/*"]
    }
  }
}

```

## Configuración de Jest

Dado que NestJS utiliza Jest como motor de pruebas unitarias y Jest no lee directamente la configuración de Typescript, tendremos que actualizar la configuración de Jest para que la herramienta pueda comprender también el mapeo de los `imports`.

Vamos al archivo `package.json` (o `jest.config.ts`) y en el apartado `jest` vamos a realizar el mismo mapeo de rutas que declaramos en la configuración de Typescript. Veamos el siguiente fragmento:

```json
"jest": {
    "rootDir": ".",
    "moduleNameMapper": {
        "^@src/(.*)$": "<rootDir>/src/$1",
        "^@common/(.*)$": "<rootDir>/src/common/$1",
        "^@modules/(.*)$": "<rootDir>/src/modules/$1"
    }
}
```

Ahora puede actualizar las líneas de `import` en los tests unitarios y podrás verificar que la nueva configuración es correcta.

```bash
yarn run test
```

Finalmente, puedes actualizar todos los `import` de la aplicación y ejecutar nuevamente el servidor y verificar el funcionamiento.

```bash
yarn run build

yarn run start

# o en modo desarrollo
yarn run start:dev
```

## Configuración de Prettier

Finalmente nos queda agregar una pequeña configuración final referente al formate de código, a cargo de `prettier` y para esta herramienta, pondremos la siguiente configuración en el archivo `.prettierrc`:

```json
{
  "trailingComma": "none",
  "singleQuote": true,
  "semi": true,
  "arrowParens": "always",
  "bracketSameLine": true,
  "proseWrap": "always",
  "tabWidth": 2,
  "printWidth": 80,
  "overrides": [{
    "files":"*.md",
    "options": {
      "printWidth": 120
    }
  }]
}
```

De esta forma hemos iniciado un nuevo proyecto con NestJS y además comenzamos a implementar buenas practicas, que mejorarán la experiencia de desarrollo y contribuirán, en general, a la higiene del proyecto.

Esto ha sido todo por esta ocasión, te invito a ver los siguientes artículos, donde vemos configuraciones de Logging, Variables de Entorno, Conexión a Bases de Datos, etc.

---

Autor: Julio César Echeverri