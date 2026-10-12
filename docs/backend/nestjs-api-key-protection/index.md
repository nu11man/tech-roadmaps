---
layout: default
title: Seguridad de rutas con Api-Key en NestJS
---

# Seguridad de rutas con Api-Key en NestJS

En un artículo anterior habíamos analizado la protección de rutas utilizando Tokens JWT. Sin embargo, esta no es la única estrategia de protección. En este artículo vamos a ver como agregar validación de llaves usando la estrategía API-KEY y, además, vamos a ver como ajustar nuestra configuración de Swagger para que este tipo de protección se vea reflejado en la documentación.

#### Contenido

- [Generar y configurar la api-key](#generar-configurar-api-key)
- [Crear un api-key Guard](#crear-api-key-guard)
- [Proteger las rutas con api-key](#proteger-rutas-con-api-key)
- [Integrar Swagger y api-keys](#integrar-swagger-api-key)

---

### Generar y configurar la api-key {#generar-configurar-api-key}

En primer lugar debemos generar una api key, esto lo logramos ejecutando la siguiente línea en nuestra terminal de comandos:

```bash
node -e "console.log(require('crypto').randomUUID())"
```

Esto nos retornará una cadena de texto como la siguiente:

```bash
d6d4f9ce-1dd2-4859-8a9f-b23bfd4bd67e
```

Ahora vamos a almacenar este cadena en nuestro archivo `.env` con el nombre `API_KEY`.
Luego, debemos actualizar el `config loader`, el `schema validation` y la interfaz que describe el modelo.

### Crear un api-key Guard {#crear-api-key-guard}

Con el secreto creado y la configuración de ambiente actualizada, vamos a crear el archivo `api-key.guard.ts` en la ruta que corresponda, por ejemplo, si vamos a proteger un módulo específico podemos crear un directorio de la forma `module_name/guards/api-key.guard.ts`. Por otra parte, si la validación la aplicaremos a los controladores de diferentes módulos, podremos guardarla en un directorio común de tipo:

```bash
src/
├── common/
│   └── guards/
│       └── api-key.guard.ts
├── auth/
├── users/
└── lists/
```

Una vez definido el archivo de nuestra clase, agregamos la siguiente implementación:

```typescript
import { timingSafeEqual } from "crypto";
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  UnauthorizedException,
} from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import { EnvConfig } from "@config/environment/env.model";

@Injectable()
export class ApiKeyGuard implements CanActivate {
  constructor(private readonly config: ConfigService<EnvConfig>) {}

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const provided = request.header("x-api-key");
    const expected = this.config.getOrThrow("api_key", { infer: true });

    if (!provided) throw new UnauthorizedException("Missing API key");

    const a = Buffer.from(provided);
    const b = Buffer.from(expected);

    if (a.length !== b.length || !timingSafeEqual(a, b)) {
      throw new UnauthorizedException("Invalid API key");
    }
    return true;
  }
}
```

Del código anterior podemos destacar cuatro aspectos muy importantes:

- Lanzamos una excepción si no hemos definido una variable de entorno de la cual podamos extraer la llave.
- Lanzamos un error `Missing API key` si el cliente no envia el header `x-api-key` (o el nombre que hayamos definido).
- Lanzamos un error `Invalid API key` si el valor enviado en la cabecera de la request no corresponde con la API Key configurada en el servidor.
- Utilizamos la función `timingSafeEqual` de la librería `crypto` para hacer la comparación de forma segura con tiempo constante, para evitar los conocidos `timing attacks`.

Cabe resaltar que como hemos hecho uso del servicio `ConfigService`, si no hemos declaro el `ConfigModule` de forma global, debemos importaren el módulo que estamos protegiendo, por ejemplo, para el módulo `Users`:

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { envSchema } from '@config/environment/env.schema';
import { configLoader } from '@config/environment/env.loader';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [configLoader],
      validationSchema: envSchema
    })
  ]
  ...
})
export class UsersModule {}
```

### Proteger las rutas con api-key {#proteger-rutas-con-api-key}

Ahora que hemos definido nuestro `api key guard` vamos a nuestro controlador del módulo de usuarios (en este ejemplo particular). Allí podemos proteger todas las rutas del controlador al aplicar el decorador `@UseGuards()` directamente sobre el mismo.

```typescript
import { UseGuards } from '@nestjs/common';
import { ApiKeyGuard } from './guards/api-key.guard';


@UseGuards(ApiKeyGuard)
@Controller('users')
export class UsersController {
  ...
}
```

Podemos también proteger rutas específicas y no el controlador completo, anotando el handler de la ruta específica, como en el siguiente ejemplo:

```typescript
export class UsersController {
  @UseGuards(ApiKeyGuard)
  @Get()
  findAll() {
    return this.usersService.findAll();
  }
}
```

### Integrar Swagger y api-keys {#integrar-swagger-api-key}

Agregar este nuevo esquema de seguridad es importante, pero no es suficiente. Debemos recordar que la calidad de una aplicación es tan alta como la calidad de su documentación, por esta razón vamos a ver como ajustar nuestra configuración de Swagger.

Swagger solo necesita saber dos cosas:

- Saber que la aplicación usa un esquema de seguridad basada en Api Key.
- Saber que recursos están protegidos por Api Key.

En primer lugar vamos a registrar el esquema en la configuración general de Swagger en el archivo `src/main.ts`:

```typescript
const config = new DocumentBuilder()
    ...
    .addApiKey(
      { type: 'apiKey', name: 'x-api-key', in: 'header' },
      'api-key'
    )
    ...
    .build();
```

El segundo parámetro del método `addApiKey` corresponde al nombre del esquema, en este caso `api-key`, es importante tenerlo presente porque es el nombre que se usa en el decorador `@ApiSecurity`.

Ahora vamos a la ruta o el controlador protegido y realizamos una anotación como la sigueinte:

```typescript
@ApiSecurity("api-key")
@UseGuards(ApiKeyGuard)
@Controller("users")
export class UsersController {}
```

Sin embargo, una implementación más limpia (menos verbosa) no llevaría a implementar un decorador compuesto:

```typescript
// src/modules/users/decorators/users-api-key.decorator.ts
import { applyDecorators, UseGuards } from "@nestjs/common";
import { ApiSecurity, ApiUnauthorizedResponse } from "@nestjs/swagger";
import { ApiKeyGuard } from "../guards/api-key.guard";

export const ApiKeyAuth = () =>
  applyDecorators(
    UseGuards(ApiKeyGuard),
    ApiSecurity("api-key"),
    ApiUnauthorizedResponse({ description: "Missing or invalid API key" }),
  );
```

Luego en el recurso protegido utilizamos el decorador `@ApiKeyAuth` que hemos creado, como por ejemplo:

```typescript
@ApiTags("users")
@ApiKeyAuth()
@Controller("users")
export class UsersController {
  @Get()
  findAll() {}
}
```

Observa ahora en la documentación generada como aparece el ícono del candado en los recursos protegidos y además, automaticamente en el dialogo de autorización, ya tenemos la entrada para indicar el api-key.

Ahora los recursos protegidos también documentan automáticamente la respuesta `HTTP 401` con el mensaje `Missing or invalid API key` que indicamos en el decorador.

Eso fue todo por esta entrada, espero que te haya resultado útil. En el próximo artículo estaremos hablando sobre la protección de recursos mediante estrategia de `limitación de requests` o `rate limiter` usando `@nestjs/throttler`.

---

Autor: Julio César Echeverri Marulanda
