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

Una vez instalado el `config module` y creados el `loader` y el `validator`, vamos a dirigirnos al archivo `app.module.ts` y en la sección de **imports** inyectamos el `config module` y su configuración como se muestra a continuación:

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

Ahora, dado que el **app.module** ha importado el `ConfigModule` podemos intentar acceder a valores de variables de entorno en la función `bootstrap` del archivo `main.ts` como se muestra en el siguiente fragmento de código, donde obtenemos los valores de `port` y el contenido del grupo `database`:

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ConfigService } from "@nestjs/config";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const configService = app.get(ConfigService);
  const port = configService.get("port");
  const database = configService.get("database");

  console.log("port", port);
  console.log("database", database);

  await app.listen(Number(port));
}

bootstrap();
```

Ahora podemos hacer uso de las variables de entorno, pero es importante tener en cuenta que tenemos dos formas de lograrlo:

#### Declaración global de ConfigModule

En este punto tenemos realmente dos formas de usar el servicio `ConfigService`. Si declaramos el módulo como **global** (con `isGlobal: true` ) en el objeto que se le pasa a `forRoot()` podemos entonces inyectar el módulo en cada servicio que lo requiera sin tener que importarlo nuevamente en cada módulo, esto es:

En el archivo `app.module.ts`

```javascript
// src/app.module.ts
ConfigModule.forRoot({
  isGlobal: true,          // 👈 add this
  load: [configLoader],
  validationSchema: envSchema,
}),
```

Luego en cualquier servicio podemos inyectar la dependencia directamente en el constructor:

```typescript
// src/users/user.service.ts
import { Injectable } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";

@Injectable()
export class UserService {
  constructor(private readonly configService: ConfigService) {}

  getDbUrl(): string {
    // Nested keys from the configLoader work with dot notation:
    return this.configService.get<string>("database.url");
  }
}
```

#### Importar ConfigModule Individualmente

En este caso vamos al módulo que requiere el acceso a variables de entorno e importamos `ConfigModule`.

```typescript
// src/users/users.module.ts
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";
import { UsersService } from "./users.service";

@Module({
  imports: [ConfigModule], // 👈 makes ConfigService injectable here
  providers: [UsersService],
})
export class UsersModule {}
```

Luego vamos al servicio e inyectamos `ConfigService` en el constructor:

```typescript
// src/users/users.service.ts
import { Injectable } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";

@Injectable()
export class UsersService {
  constructor(private readonly configService: ConfigService) {}

  getDbUrl(): string {
    // Nested keys from your configLoader work with dot notation:
    return this.configService.get<string>("database.url");
  }
}
```

---

### Otras Buenas Prácticas

#### Type Safety

Al acceder una variable de entorno de la siguiente forma:

```typescript
this.configService.get<string>("port");
```

El valor de retorno es `string | undefined` si queremos que se lance una excepción si la variable no está definida podemos usar el método `getOrThrow`:

```typescript
this.configService.getOrThrow<string>("database.url");
```

Sin embargo, desde secciones previas ya contamos con el validador de objetos `Joi` para asegurarnos que realmente no habrán, en tiempo de ejecución, variables indefinidas.

#### Parseado de Valores en el Config Loader

Dado que todos los valores que vienen de un archivo de ambiente son `strings`, se pueden _parsear_ los valores al momento de usarlos en el servicio o al momento de cargarlos en el _ConfigLoader_ definido previamente:

Transformar un valor para usarlo:

```typescript
const port = configService.get("port");

await app.listen(Number(port));
// o
await app.listen(parseInt(port));
```

O hacer el _parsing_ en el **ConfigLoader**.

```typescript
export const configLoader = () => {
  return {
    port: parseInt(process.env.PORT, 10) || 3000,
  };
};
```

---

Esto ha sido todo por hoy, espero que te resulte interesante el contenido de este artículo.

Autor: Julio Echeverri
