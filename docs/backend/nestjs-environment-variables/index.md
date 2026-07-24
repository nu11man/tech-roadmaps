---
layout: default
title: Variables de Entorno en NestJS
---

# Variables de Entorno en NestJS

Una de las prácticas más importantes en el desarrollo de software es la gestión de llaves, secretos y valores de configuración requeridos por la aplicación o el servidor al momento de compilar o en tiempo de ejecución.

Cuando trabajamos con NestJS solemos almacenar los valores de estas variables de configuración (también llamadas variables de ambiente) en archivos de texto que usualmente llamamos archivos de ambiente, donde **"ambiente"** identifica los posibles entornos de trabajo del software, pueden ser _Producción_, _Stage_, _Load_, _Test/QA_, etc.

---

La gestión de las variables de entorno la hacemos a través un módulo de NestJS llamado `ConfigModule` que instalamos con la siguiente línea.

```bash
npm install --save @nestjs/config
```

Antes de crear la configuración vamos a necesitar un **loader** (una función que indica como se cargarán las variables, por ejemplo agrupadas o individualmente) y un **schema** de validación para asegurarnos siempre de contar con las variables de entorno completas al momento de iniciar el servidor.

### Cargar las Variables de Entorno (Config Loader)

En una carpeta de configuración podemos crear un archivo con nombre `config-loader.ts` y en este archivo exportamos una función que se encarga de retornar un objeto que mapea todos los campos del archivo `.env`. El código se presenta a continuación:

```javascript
export const configLoader = () => {
  return {
    port: process.env.PORT,
    database: {
      url: process.env.DATABASE_URL,
      username: process.env.DATABASE_USERNAME,
      password: process.env.DATABASE_PASSWORD,
    },
  };
};
```

### Validación de Variables de Entorno (Schema Validation)

Para validar las variables de entorno vamos a requerir un validador de objetos como `joi` que ya cuenta con integración directa con NestJS.

```bash
npm install --save joi
```

Vamos a crear un archivo de `schema` en la carpeta de configuraciones llamado `env-schema.ts`. En este archivo vamos a definir un objeto que describa el `schema` del contenido de nuestro archivo de ambiente, es decir, cuales variables deben estar definidas y que tipo de valor es el esperado:

```javascript
import * as Joi from "joi";

export const envSchema = Joi.object({
  PORT: Joi.string().required(),
  DATABASE_URL: Joi.string().required(),
  DATABASE_USERNAME: Joi.string().required(),
  DATABASE_PASSWORD: Joi.string().required(),
});
```

### Configuración del servicio

Una vez instalamos el `config module` y que creamos el `loader` y el `validator`, vamos a dirigirnos al archivo `app.module.ts` y en la sección de **imports** inyectamos el `config service` y su configuración como se muestra a continuación:

```javascript
import { ConfigModule } from "@nestjs/config";
import { configLoader } from "./config/envLoader";
import { envSchema } from "./config/envSchema";

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [configLoader],
      validationSchema: envSchema,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### Cómo Usar las Variables de Entorno
