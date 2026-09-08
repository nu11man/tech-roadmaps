---
layout: default
title: Autenticación de Usuarios con PassportJS y NestJS
---

# Autenticación de Usuarios con PassportJS

Un aspecto fundamental en los servicios web es la posibilidad de controlar el acceso y las acciones que pueden realizar los usuarios. En esta entrada vamos a ver como implementar un flujo básico de autenticación de usuarios (una estrategia local) usando el paquete _PassportJS_.

#### Contentido

- [Instalación de PassportJS](#instalacion-passport)
- [Móulo de autenticación](#modulo-de-autenticacion)
- [Validación de email-password](#validacion-user-password)
- [Creación de Local Strategy](#creacion-local-strategy)
- [Creación de endpoint de login](#creacion-endpoint-login)
- [Estado final de AuthModule](#auth-module)

---

### Instalación de PassportJS {#instalacion-passport}

PassportJS es uno de los paquetes más utilizados por la comunidad de NodeJS para la autenticación de usarios, permite la autenticación a través de redes sociales o usuario y contraseña, siendo esta última la más básica y la que implementaremos en este artículo a través de la entrategia `local`.

Para instalar este paquete en nuestro proyecto vamos a ejecutar la siguiente línea si usas `npm`:

```bash
npm install @nestjs/passport passport passport-local

npm install --save-dev @types/passport-local
```

O si usas `yarn` podemos ejecutar lo siguiente:

```bash
yarn add @nestjs/passport passport passport-local

yarn add --dev @types/passport-local
```

_Nota_: `passport-local` es una dependencia adicional que usamos si vamos a implementar una autenticación básica de usario-contraseña.

### Módulo de autenticación {#modulo-de-autenticacion}

Lo primero que debemos hacer para una correcta implementación es crear un módulo (y servicio) dedicado a la autenticación. Podemos apoyarnos en el asitente de NestJS para esta tarea.

```bash
nest g module auth

nest g service auth
```

El siguiente paso es agregar un método en nuestro servicio de usuarios que nos permite obtener la información de un usuario dado su email.

```typescript
async findUserByEmail(email: string) {
  const user = await this.userRepository.findOne({
    where: { email }
  });
  return user;
}
```

A diferencia de los métodos que podemos tener en el servicio de usurios aquí no requerimos el modelo completo de las relaciones a otras tablas como Post o Perfil.

Luego, para poder hacer uso de este método en nuestro módulo de autenticación debemos hacer un par de ajustes. Debemos exportar el servicio `UsersService` en nuesto módulo `UsersModule`:

```typescript
@Module({
  ...
  exports: [UsersService]
})
export class UsersModule {}
```

Después debemos ir a nuestro módulo `AuthModule` e importar el módulo `UsersModule`:

```typescript
import { UsersModule } from "@src/users/users.module";

@Module({
  imports: [UsersModule],
})
export class AuthModule {}
```

De esta forma nuestro módulo de autenticación tendrá acceso a todos los `exports` expuestos por el módulo de usuarios, podremos acceder a los métodos del servicio con la siguiente inyección de dependencias:

```typescript
import { Injectable } from '@nestjs/common';
import { UsersService } from '@src/users/users.service';

@Injectable()
export class AuthService {
  constructor(private readonly usersService: UsersService) {}

  ...
}
```

### Validación de email-password {#validacion-user-password}

Ahora en nuestro `AuthService` vamos a crear un método llamado `validateUser` que recibirá el email y password, y se encargará de validar que el usuario exista y que la contraseña recibida sea correcta. Ese método tiene la siguiente implementación:

```typescript
@Injectable()
export class AuthService {
  constructor(private readonly usersService: UsersService) {}

  async validateUser(email: string, password: string) {
    const user = await this.usersService.findUserByEmail(email);

    if (!user) {
      throw new UnauthorizedException("Unauthorized");
    }

    const isMatch = await bcrypt.compare(password, user.password);

    if (isMatch) {
      return user;
    }

    throw new UnauthorizedException("Unauthorized");
  }
}
```

En primer lugar, verificamos si el usuario existe en nuestra base de datos (usando el email), si existe, entonces validamos la contraseña (hasheada), si email y password son correctos retornamos el usuario, si no, levantamos una excepción.

### Creación de Local Strategy {#creacion-local-strategy}

Ahora que en `AuthService` tenemos un método para validar la identidad del usuario, vamos a implementar una de las muchas estrategias que nos propone Passport, la autenticación de usuario y contraseña.

En primer lugar, dentro del módulo `auth` vamos a crear la carpeta `strategies` que agrupará todos los archivos que definen las estrategias que construyamos, por ahora vamos a crear en esta carpeta el archivo `local.strategy.ts` y vamos a depositar allí el siguiente contenido:

```typescript
import { Strategy } from "passport-local";
import { PassportStrategy } from "@nestjs/passport";
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { AuthService } from "../auth.service";

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy, "local") {
  constructor(private authService: AuthService) {
    super({
      usernameField: "email",
      passwordField: "password",
    });
  }

  async validate(email: string, password: string) {
    const user = await this.authService.validateUser(email, password);
    return user;
  }
}
```

Del código anterior debemos observar que:

- PassportStrategy recibe como segundo parámetro el nombre con el que referenciaremos a la estrategia en los `Guards` y otros lugares.
- Dado que el nombre por defecto de los campos de la estrategia es `userName` y `password`, debemos actualizar en el constructor de la clase `PassportStrategy` los nombres que correspondan.
- Dado que el manejo de excepciones lo hicimos en el método `validateUser` del `authService`, aquí podemos obviar la gestión de excepciones.

### Creación de endpoint de login {#creacion-endpoint-login}

Con la estrategia local implementada, debemos ahora crear una ruta de `login` para que nuestros usuarios puedan enviar las credenciales y poder validarlas.

Vamos a generar un _Controller_ con el asistente de NestJS como lo hicimos en nuestro artículos sobre controladores, handlers y services.

```bash
nest g controller auth
```

Luego con el controlador generado, creamos una ruta específica de login como se ve en el siguiente fragmento de código:

```typescript
import { Controller, Post, Req, UseGuards } from "@nestjs/common";
import type { Request } from "express";
import { AuthService } from "./auth.service";
import { AuthGuard } from "@nestjs/passport";

@Controller("auth")
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @UseGuards(AuthGuard("local"))
  @Post("login")
  login(@Req() req: Request) {
    return req.user;
  }
}
```

De este código destacamos lo siguiente:

- Utilizamos el decorador `@UseGuards` no para protejer la ruta sino para ejecutar una función que implementa la autenticación con la estrategia que nombramos `local`.
- El nombre `local` debe coincidir con el nombre que le dimos a la estrategia en el archivo `local.strategy.ts`.
- Utilizamos el decorador `@Req` de `@nestjs/common` y el tipo `Request` de `express` para mejorar un poco la experiencia de desarrollo con Typescript.
- Dado que el objeto retornado por la estrategia implementada con Passport queda agregado al cuerpo de la request, podamos hacer uso de ese objeto en el controlador (`req.user`).

### Estado final de AuthModule {#auth-module}

Finalmente demos una mirada al estado actual del módulo de autenticación `AuthModule`.

```typescript
import { Module } from "@nestjs/common";
import { AuthService } from "./auth.service";
import { UsersModule } from "@src/users/users.module";
import { LocalStrategy } from "./strategies/local.strategy";
import { PassportModule } from "@nestjs/passport";
import { AuthController } from "./auth.controller";

@Module({
  imports: [UsersModule, PassportModule],
  providers: [AuthService, LocalStrategy],
  controllers: [AuthController],
})
export class AuthModule {}
```

Finalmente, si ejecutamos nuestro servidor y comenzamos a lanzar peticiones a nuestro endpoint de `login` vamos a encontrar que:

- Si el email o la contraseña no son correctos, la respuesta será un error de usuario no autorizado.
- Si el email y la contraseña son correctos, el endpoint nos retornará una respuesta con los datos del usuario.

Esta es una implementación elemental. Sin embargo, el paso siguiente es hacer que este endpoint no solo retorne la información del usuario cuando la autenticación fue exitosa, sino que genere y envíe en la respuesta un Token JWT que permita al usuario acceder a recursos protegidos.

Nos vemos en el próximo artículo donde creamos el soporte para la generación de Tokens `JWT` y vamos a configurar la protección de rutas usando PassportJS.

---

Autor: Julio César Echeverri
