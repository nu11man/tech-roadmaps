---
layout: default
title: Iniciar un Proyecto React con Vite
---

# Iniciar un Proyecto React con Vite

_Vite_ es una herramienta de compilación y un servidor de desarrollo que funciona excepcionalmente bien con proyectos React, Vue, Svelte, entre otros. Cuenta actualmente con una gran comunidad que lo respalda y podemos decir que fue el sucesor del anteriormente famoso **create-react-app**.

En esta entrada vamos a ver como crear y configurar un nuevo proyecto de React (con soporte de Typescript) utilizando _Vite_.

## Iniciar el proyecto

Para crear un nuevo proyecto utilizando vite el formato del comando que debemos escribir es el siguiente:

```bash
yarn create vite my-app-name [--template name]
```

Por lo tanto, si queremos crear un proyecto React con soporte de Typescript, el comando que debemos ejecutar es:

```bash
# usando yarn
yarn create vite my-app-name --template react-ts

# usando npm
npm create vite@latest my-app-name -- --template react-ts
```

Después de ejecutar este comando, puedes ingresar a la carpeta del proyecto, instalar las dependencias y ejecutar el servidor de desarrollo. Estos tres pasos corresponden a las siguientes instrucciones en la terminal:

```bash
cd my-app-name
yarn
yarn dev
```

**Nota:** Es posible utilizar un punto (`.`) como nombre de la aplicación en el comando anterior si queremos que la aplicación se implemente en el directorio actual.

## Configuración de Typescript y Aliases

La configuración por defecto proporcionada por Vite viene bien, pero vamos a activar algunas flags que no vienen mal a la salud del proyecto. Puedes copiar y pegar la siguiente configuración en el archivo `tsconfig.app.json`.

```json
{
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
    "target": "es2023",
    "lib": ["ES2023", "DOM"],
    "module": "esnext",
    "types": ["vite/client"],
    "allowArbitraryExtensions": true,
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "erasableSyntaxOnly": true,
    "noFallthroughCasesInSwitch": true,

    /* Alias */
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@assets/*": ["./src/assets/*"]
    }
  },
  "include": ["src"]
}
```

En la sección `Alias` del archivo anterior puedes encontrar ejemplos de como configurar aliases de rutas. Esto es importante para la legibilidad y mantenibilidad del proyecto, porque nos permite pasar de `imports` relativos como:

```typescript
import SearchBar from "../../../../components/SearchBar";
```

A imports más concisos como:

```typescript
import SearchBar from "@components/SearchBar";
```

La configuración descrita en el archivo _tsconfig_ es suficiente para indicar a Typescript donde encontrar los archivos de código fuente cuando los importamos utilizado alias, pero aún debemos indicarle a Vite como se resuelven estos imports. Para lograr esto, necesitamos ajustar la configuración del archivo `vite.config.ts`.

En el archivo `vite.config.ts`, especificamente en el objeto pasado a la función `defineConfig` vamos a agregar el atributo `resolve` y allí mapear los alias que deseamos definir (es importante manterlos alineados con los aliases definidos en el archivo de configuración de Typescript). Tomemos como ejemplo el siguiente código.

```javascript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

// https://vite.dev/config/
export default defineConfig({
  plugins: [react()],
  resolve: {
    //agrega este atributo
    alias: {
      "@": path.resolve(__dirname, "src"),
      "@components": path.resolve(__dirname, "src/components"),
      "@assets": path.resolve(__dirname, "src/assets"),
    },
  },
});
```

**Nota:** Es posible que el editor muestre un error donde indica que no encuentra el módulo `path` o sus definiciones de tipo, para ello debemos instalar `@types/node` y esto lo logramos con el siguiente comando:

```bash
yarn add --dev @types/node
```

Ahora puedes ejecutar nuevamente el servidor y hacer uso de los **import alias** y continuar con el desarrollo de tu aplicación web.

---

Esto ha sido todo por hoy, espero que esta entrada te haya resultado útil.

Nos leemos en un próximo artículo.

Autor: Julio Echeverri.
