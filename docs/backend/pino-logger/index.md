---
layout: default
title: NestJS Logging con Pino Logger
---

# NestJS Logging con Pino Logger

Un **log** se puede entender como un **registro** o una **traza** que se deja en algún medio, puede ser un mensaje en archivo de texto, una entrada en una base de datos, etc. El objetivo del logging (la acción de registrar) es mostrar mensajes que ayudan a entender que ocurre mientras un software ejecuta sus tareas.

Un **logger** es entonces la herramienta que se utiliza para escribir en los registros (escribir en los **logs**).

NestJS cuenta con un **logger** “**de fábrica**” pero para proyectos que escalan es importante recurrir a herramientas más profesionales. En esta entrada vamos a _instalar_, _configurar_ y _usar_ el logger _Pino_, uno de los más populares en el entorno de NestJS.

**Contenido**

- [Instalación de dependencias](#instalacion-dependencias)
- [Coiguración de Pino Logger](#configuracion-de-pino-logger)
  - [Logging Format y Correlation ID](#logging-format-and-correlation-id)
  - [Inyectar Pino en la aplicación](#inyectar-pino-en-aplicacion)
- [Cómo usar el logger](#como-usar-el-logger)

---

## Instalación de dependencias {#instalacion-dependencias}

En primer lugar vamos a instalar las dependencias:

```bash
yarn add nestjs-pino pino-http

# o usando npm

npm install --save nestjs-pino pino-http
```

Como dependencia de desarrollo agregaremos el paquete `pino-pretty` para ver los logs de forma más legible en la terminal, mientras estamos desarrollando localmente:

```bash
yarn add --dev pino-pretty

# o usando npm

npm install --save-dev pino-pretty
npm install -D pino-pretty
```

## Configuración de Pino Logger {#configuracion-de-pino-logger}

Podemos comenzar a leer logs de Pino con una configuración muy básica pero en esta ocasión vamos a ir un poco más allá y configurar algunos aspectos específicos que posteriormente se pueden ir puliendo en mayor medida según las necesidades.

### Logging Format y Correlation ID {#logging-format-and-correlation-id}

Cuando trabajamos del lado del **backend** es de suma importancia poder hacer seguimiento a las peticiones que procesamos, poder trazar el camino que siguen y cómo se relacionan, para esto nos valemos de una propiedad que denominamos correlation-id que, como su nombre indica asigna un identificador de correlación a cada petición.

Para configurar el logger vamos a crear un archivo en nuestra carpeta de configuraciones generales, por ejemplo, `src/config/logging.config.ts` y agregaremos el siguiente contenido:

```typescript
// src/config/logging.config.ts

import { randomUUID } from "crypto";
import { IncomingMessage, ServerResponse } from "http";

export const CORRELATION_ID_HEADER = "x-correlation-id";

export const PINO_CONFIG = {
  pinoHttp: {
    transport:
      process.env.NODE_ENV === "production"
        ? undefined
        : {
            target: "pino-pretty",
            options: {
              messageKey: "message",
            },
          },
    messageKey: "message",
    genReqId: (req: IncomingMessage, res: ServerResponse) => {
      const incomingCorrelationId = req.headers[CORRELATION_ID_HEADER] as
        | string
        | undefined;
      const correlationId = incomingCorrelationId ?? randomUUID();
      req.headers[CORRELATION_ID_HEADER] = correlationId;
      res.setHeader(CORRELATION_ID_HEADER, correlationId);
      return correlationId;
    },
    customProps: (req: IncomingMessage) => ({
      correlationId: req.headers[CORRELATION_ID_HEADER] || "N/A",
    }),
    autoLogging: false,
    serializers: {
      req: () => {
        return undefined;
      },
      res: () => {
        return undefined;
      },
    },
  },
};
```

Con este archivo estamos indicando 4 aspectos fundamental:

- Solo utilizamos `pino-pretty` en ambientes no productivos.
- Agregamos el **correlation-id** como header de nuestras requests y responses.
- En `customProps` seleccionamos el **correlation-id** como parámetro para imprimir en el log.
- En cada entrada de log se imprime como una _custom property_ el correlation-id de la petición.
- Deshabilitar el logging automático de Pino ya que muestra mucho contenido que no necesariamente deseamos ver en los logs.

### Inyectar Pino en la Aplicación {#inyectar-pino-en-aplicacion}

Vamos al archivo `app.module.ts` e importamos `LoggerModule` de pino:

```typescript
// app.module.ts
import { LoggerModule } from 'nestjs-pino'
import { PINO_CONFIG } from './config/logging/logging.config'

...

@Module({
  imports: [LoggerModule.forRoot(PINO_CONFIG)],
  controllers: [AppController],
  providers: [AppService],
})

export class AppModule {}
```

Posteriormente en el archivo `main.ts` vamos a modificar la función `bootstrap()` para indicar a NestJS que debe usar una instancia de **Pino-Logger** en lugar del logger por defecto:

```typescript
// main.ts
import { Logger } from 'nestjs-pino';

...

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useLogger(app.get(Logger));
  ...
}
```

Al reiniciar la aplicación veremos que ya no estamos usando el logger de NestJS sino el logger de Pino con el formato configurado.

## Cómo usar el Logger {#como-usar-el-logger}

Después de la configuración vamos a usar el `Logger` en nuestros servicios, módulos, etc. Para ello, creamos una instancia del logger como miembro privado de la clase, le pasamos al constructor el nombre de la clase desde donde estamos instanciando el logger para agregarlo al contexto y, luego utilizamos su método `.log`

Es importante tener en cuenta que importaremos el logger desde `@nestjs/common`. Esto porque si a futuro cambiamos el paquete de logging, no tengamos que reemplazarlo en todos los archivos que lo usen, sino en la configuración global.

```typescript
import { Logger } from '@nestjs/common';

// En la clase de servicio agregamos el logger como una nueva propiedad
private readonly logger = new Logger(NombreClase.name)

...
// En los métodos lo usamos como:
this.logger.log("Contenido para el log")
```

---

Espero que esta entrada te haya resultado útil, nos leemos en un próximo artículo.

Autor: Julio Echeverri.
