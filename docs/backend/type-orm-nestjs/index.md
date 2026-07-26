---
layout: default
title: Configuración de TypeORM en NestJS
---

# Configuración de TypeORM en NestJS

Un ORM es un xxx que nos permite realizar un mapeo de las entidades de base de datos a objetos en el lenguaje que estemos utilizando, lo que permite un manejo más flexible de cara al desarrollador al abstraer en gran medida la carga de la gestión de la base de datos desde nuestro código fuente.

En NestJS, contamos con múltiples alternativas para realizar este mapeo automáticamente, siendo _TypeORM_ uno de los más utilizados por la comunidad ya que, además, tiene soporte nativo para trabajar con bases de datos relacionales y no relacionales (SQL y NoSQL), por ejemplo, MySQL, PostgreSQL, MongoDB, etc.

## Instalación

Vamos a configurar TypeORM en nuestro proyecto, suponiendo además que trabajaremos con PostgreSQL como base de datos. Entonces realizaremos la instalación de 3 dependencias.

1. TypeORM (`typeorm`)
2. El módulo de TypeORM para NestJS (`@nestjs/typeorm`)
3. El driver de la base de datos, en este caso PostgreSQL (`pg`)

Ejecutamos la siguiente línea en la terminal de comandos:

```bash
npm install --save @nestjs/typeorm typeorm pg
```

Una vez se finaliza la instalación podemos continuar con la configuración de la conexión a la base de datos.

## Configuración

En primer lugar vamos a crear un archivo en el directorio `src/config/database` cuyo nombre puede ser `typeorm.config.ts`. Allí definimos una función que recibe como parámetro el config service (una dependencia inyectada) y retorna un objeto de configuración de TypeORM:

```typescript
import { ConfigService } from "@nestjs/config";
import { TypeOrmModuleOptions } from "@nestjs/typeorm";

export const typeOrmConfig = (
  configService: ConfigService,
): TypeOrmModuleOptions => ({
  type: "postgres",
  host: configService.getOrThrow<string>("database.host"),
  port: configService.getOrThrow<number>("database.port"),
  username: configService.getOrThrow<string>("database.username"),
  password: configService.getOrThrow<string>("database.password"),
  database: configService.getOrThrow<string>("database.name"),

  autoLoadEntities: true,
  // Dev convenience only — never auto-sync schema in production.
  synchronize: configService.get<string>("NODE_ENV") !== "production",
});
```

Para indicar a nuestro servidor como debe conectarse a la base de datos, vamos al archivo `app.module.ts` e importamos el módulo TypeORM y lo inyectamos en el módulo principal de la aplicación. Utilizamos la función de configuración que creamos anteriormente:

```typescript
import { TypeOrmModule } from "@nestjs/typeorm";
import { typeOrmConfig } from "./config/database/typeorm.config";

@Module({
  imports: [
    ...TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: typeOrmConfig,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
