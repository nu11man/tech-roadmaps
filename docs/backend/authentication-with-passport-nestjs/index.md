---
layout: default
title: Autenticación de Usuarios con PassportJS y NestJS
---

# Autenticación de Usuarios con PassportJS

Un aspecto fundamental en los servicios web es la posibilidad de controlar el acceso y las acciones que pueden realizar los usuarios. En esta entrada vamos a ver como implementar un flujo básico de autenticación de usuarios (una estrategia local) usando el paquete _PassportJS_.

#### Contentido

- [Instalación de PassportJS](#instalacion-passport)
- [Móulo de autenticación](#modulo-de-autenticación)
- [Validación de email-password](#validacion-user-password)
- [](#)
- [](#)

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

### Módulo de autenticación {#modulo-de-autenticación}

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
