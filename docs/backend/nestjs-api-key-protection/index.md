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

Del código anterior podemos destacar tres aspectos muy importantes:

- Lanzamos un error `Missing API key` si no hemos definido una variable de entorno de la cual podamos extraer la llave.
- Lanzamos un error `Invalid API key` si el valor enviado en la cabecera de la request no corresponde con la API Key configurada en el servidor.
- Utilizamos la función `timingSafeEqual` de la librería `crypto` para hacer la comparación de forma segura con tiempo constante, para evitar los conocidos `timing attacks`.

### Proteger las rutas con api-key {#proteger-rutas-con-api-key}

Ahora que hemos definido nuestro `api key guard` vamos a ir a nuestro controlador del módulo de usuarios (en este ejemplo particular).

### Integrar Swagger y api-keys {#integrar-swagger-api-key}

---

Autor: Julio César Echeverri Marulanda

Fuente: https://claude.ai/chat/d64ff79a-24d4-442d-b75e-9cb7a2d5689c
