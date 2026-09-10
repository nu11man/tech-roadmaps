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
    "email": "gorditos-byd@outlook.com",
    "profile": {
      "id": 6,
      "name": "Julio César",
      "lastName": "Echeverri Marulanda",
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

### Protección de endpoints {#proteccion-de-endpoints}

Ya tenemos un endpoint de `/login` que le entrega al cliente un access token cuando la autentición se realizó correctamente y una _strategy_ de PassportJS que realiza la validación de tokens JWT que se envían en el header de una petición. Ahora vamos a ajustar el proyecto para proteger algunos endpoints de interés.
