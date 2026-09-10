---
layout: default
title: Autenticación basada en token JWT
---

# Autenticación basada en token JWT

En el artículo anterior implementamos la autenticación de los usuarios usando la una _Local Strategy_ de PassportJS, esa local strategy particularmente se basa en la verificación de _email_ y _password_. Eso está muy bien, implementamos el endpoint de `/login` y si el email y la contraseña son correctos, retornamos la información del usuario. Sin embargo, nos hacen falta dos aspectos muy importantes.

- Un mecanismo que permita al usuario acceder a recursos protegidos sin tener que enviar constantemente el email y password.
- Implementar protección en diferentes recursos que no deben ser accedidos por entidades no autenticadas.

Para resolver estos dos aspectos, vamos a ver como podemos utilizar la especificación JWT (JSON Web Token) para entregarle al usuario el equivalente a una llave digital, y además implementaremos la protección de algunos endpoints.

#### Contenido

- [Instalación de dependencias](#instalacion-de-dependencias)
- [Generación de tokens JWT](#generacion-de-tokens-jwt)
- [Creación de JWT Strategy](#jwt-strategy)
- [Protección de endpoints](#proteccion-de-endpoints)
- [Acceder al contenido de JWT](#acceder-contenido-jwt)

---

### Instalación de dependencias {#instalacion-de-dependencias}

Lo primero que debemos hacer es instalar las dependencias para trabajar con JWT, para ello ejecutamos las siguientes líneas.

```bash
npm install --save @nestjs/jwt passport-jwt
npm install --save-dev @types/passport-jwt
```

O si usamos `yarn`:

```bash
yarn add @nestjs/jwt passport-jwt
yarn add --dev @types/passport-jwt
```

### Generación de tokens JWT {#generacion-de-tokens-jwt}

Vamos a iniciar ahora a trabajar en la creación de JSON Web Tokens, para ello lo primero que debemos hacer es importar en nuestro módulo `AuthModule` el módulo `JwtModule`:

```typescript
import { JwtModule } from "@nestjs/jwt";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { EnvConfig } from "@src/config/environment/env.model";

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService<EnvConfig>) => ({
        secret: configService.get("jwtSecret", { infer: true }),
        signOptions: { expiresIn: "1h" },
      }),
    }),
  ],
  providers: [AuthService, LocalStrategy],
  controllers: [AuthController],
})
export class AuthModule {}
```

Observemos que en el código anterior aplicamos directamente el `ConfigService` para obtener el secreto desde el archivo de ambiente.

Ahora veamos un pequeño truco para generar secretos que podemos considerar seguros, ejecutemos la siguiente línea en la terminal de comandos:

```bash
# Genera una cadena aleatoria de 64 caracteres
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

El resultado de la línea anterior será algo como esto:

```bash
6640642c7023c4241a41780989b27a2ab0c1373bc23b7bfc8116c644bedd7bc1d710a19d916b78743ae888c9d001cbe0c89a244dd05f91dd7bed155c9b14ba18
```

Ahora para crear el token vamos a definir el método `generateToken` en el servicio `AuthService`. Allí aplicaremos los siguientes cambios:

```typescript
import { JwtService } from '@nestjs/jwt';
import { User } from '@src/users/entities/user.entity';

@Injectable()
export class AuthService {
  constructor(
    private readonly jwtService: JwtService
  ) {}

  ...

  generateToken(user: User) {
    const payload = { sub: user.id };
    return this.jwtService.sign(payload);
  }
}
```

Cabe anotar que el payload del JWT puede contener campos arbitrarios, aunque la especificación contiene casi todos los atributos que podríamos llegar a necesitar, por lo que vale la pena dar una revisada y saber que podemos agregar.

Como ya podemos generar tokens JWT en el serivicio `AuthService`, vamos a ir al controlador `AuthController` y vamos a hacer el siguiente ajuste en el endpoint de `/login`, para retornar al usuario el access token con el que podrá acceder a los recursos protegidos por autenticación.

```typescript
import { User } from "@src/users/entities/user.entity";

@Controller("auth")
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @UseGuards(AuthGuard("local"))
  @Post("login")
  login(@Req() req: Request) {
    const user = req.user as User;
    return {
      user: user,
      access_token: this.authService.generateToken(user),
    };
  }
}
```

Observa que ahora la enviar una petición con email y password correctos, la respuesta es la sigueinte:

```json
{
  "user": {
    "id": 6,
    "email": "example.user@outlook.com",
    "profile": {
      "id": 6,
      "name": "John",
      "lastName": "Doe",
      "avatarUrl": "https://myphoto.com/julio.jpg",
      "createdAt": "2026-09-10T04:58:22.538Z",
      "updatedAt": "2026-09-10T04:58:22.538Z"
    },
    "createdAt": "2026-09-10T04:58:22.538Z",
    "updatedAt": "2026-09-10T04:58:22.538Z"
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOjYsImlhdCI6MTc4OTAxNjM4OSwiZXhwIjoxNzg5MDE5OTg5fQ.FjrZZldWz9QuScXpnVJ299xS6W5qUokTvITZrOtlkyU"
}
```

### Creación de JWT Strategy {#jwt-strategy}

Para validar la autenticidad o integridad de un jwt token que viene en el header de una petición a uno de nuestros endpoints, vamos a implementar una Passport Strategy de JWT. Para ello vamos a crear una archivo de nombre `jwt.strategy.ts` en nuestro directorio de estrategias `src/auth/strategies/`.

```typescript
import { ExtractJwt, Strategy } from "passport-jwt";
import { PassportStrategy } from "@nestjs/passport";
import { Injectable } from "@nestjs/common";
import { AuthService } from "../auth.service";
import { ConfigService } from "@nestjs/config";
import { EnvConfig } from "@src/config/environment/env.model";

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy, "jwt") {
  constructor(
    configService: ConfigService<EnvConfig>,
    private readonly authService: AuthService,
  ) {
    const jwtSecret = configService.get("jwtSecret", { infer: true });

    if (!jwtSecret) {
      throw new Error("JWT secret is not configured");
    }

    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: jwtSecret,
    });
  }

  validate(payload: { sub: string }) {
    return { userId: payload.sub };
  }
}
```

Observa que en esta estrategia obtenemos nuevamente el secreto con el que se firman los tokens JWT al momento de generarlos, pero en este caso se usa para validarlos. También tenemos un método `validate` que se ocupa de obtener el contenido de la carga útil del JWT. En este método también podríamos realizar una consulta a la base de datos para retornar más información sobre el usuario y no solo la que está presente en el token.

Un ejemplo de un método `validate` alternativo se muestra en el siguiente código:

```typescript
async validate(payload: { sub: string }) {
  const user = await this.authService.validateUserById(payload.sub);
  if (!user) {
    throw new UnauthorizedException('Unauthorized');
  }
  return user;
}
```

Como conclusión, tengamos en cuenta que el valor retornado por el método `validate` se agrega al objeto request en el atributo `user` para que el manejador o los servicios que usamos en el controller tengan acceso directo a la información validada.

Finalmente no podemos olvidar que como hemos creado una nueva estrategia, debemos ajustar nuestro módulo `AuthModule` con este nuevo provider:

```typescript
import { JwtStrategy } from './strategies/jwt.strategy';

@Module({
  imports: [...],
  providers: [AuthService, LocalStrategy, JwtStrategy],
  controllers: [AuthController]
})
```

### Protección de endpoints {#proteccion-de-endpoints}

Ya tenemos un endpoint de `/login` que le entrega al cliente un _access token_ cuando la autentición se realizó correctamente y una _strategy_ de PassportJS que realiza la validación de tokens JWT que se envían en el header de una petición. Ahora vamos a ajustar el proyecto para proteger algunos endpoints de interés.

Cuando llega el momento de proteger una ruta específica, se puede hace por endpoint o por controller. Proteger un endpoint específico se ve así:

```typescript
import { UseGuards } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Controller('posts')
export class PostsController {
  constructor(private readonly postsService: PostsService) {}

  @UseGuards(AuthGuard('jwt'))
  @Post()
  create(@Body() createPostDto: CreatePostDto) {
    return this.postsService.create(createPostDto);
  }
  ...
}
```

Mientras que proteger todas las rutas de un _Controller_ se ve así:

```typescript
import { UseGuards } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@UseGuards(AuthGuard('jwt'))
@Controller('posts')
export class PostsController {
  constructor(private readonly postsService: PostsService) {}
  ...
}
```

Observa como usamos el decorador `@UseGuards` y la función `AuthGuard` de Passport a la que le pasamos el nombre de la _Strategy_ que implementamos para validar los tokens JWT.

### Acceder al contenido de JWT {#acceder-contenido-jwt}

Hasta este punto validamos la autenticidad de los JSON Web Tokens que el cliente nos envía, pero en algunos casos es probable que necesitemos acceder al contenido del Token dentro de nuestros controladores o servicios.

En primer lugar viene bien crear un tipo de dato que nos indique el contenido (payload) del JSON Web Token. El archivo `auth/models/payload.model.ts` podría contener lo siguiente:

```typescript
export interface Payload {
  sub: number;
}
```

El método `validate` de la estrategía `JwtStrategy` podría directamente retornar el payload.

```typescript
validate(payload: Payload) {
  return payload;
}
```

Finalmente, en el controlador podríamos acceder al contenido del payload (almacenado automáticamente) como sigue:

```typescript
import { AuthGuard } from '@nestjs/passport';
import type { Request } from 'express';
import { Payload } from '@src/auth/models/payload.model';

@Controller('posts')
export class PostsController {
  constructor(private readonly postsService: PostsService) {}

  @UseGuards(AuthGuard('jwt'))
  @Post()
  create(@Body() createPostDto: CreatePostDto, @Req() req: Request) {
    const user = req.user as Payload;
    const userId = user.sub;
    return this.postsService.create(createPostDto, userId);
  }
  ...
}
```

En el ejemplo anterior sacamos el userId del DTO de creación para extraerlo del JSON Web Token y que el cliente no tenga que enviarlo explícitamente en la request, por lo tanto debemos extraerlo del JWT y pasarlo como un prámetro adicional en el servicio.

Esto ha sido todo por hoy, en el próximo artículo vamos a abordar la generación automática de documentación con `Swagger`, no te lo pierdas.

---

Autor: Julio César Echeverri
