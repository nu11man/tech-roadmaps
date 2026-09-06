---
layout: default
title: Excluir atributos de respuesta con Class Transformer
---

# Excluir atributos de respuesta con Class Transformer

En las entradas anteriores hicimos un extensivo uso del paquete `class-validation` que como su nombre indica, nos permite realizar validaciones de objetos, que normalmente corresponden a objetos enviados a nuestros servicios, los que llamamos DTOs.

Sin embargo, también hay otro caso de uso muy común y es el evitar enviar campos innecesarios o sensibles en nuestras respuestas hacia el cliente, como por ejemplo, contraseñas.

En esta entrada veremos como activar el filtrado automático de atributos en nuestras respuestas usando el paquete `class-transformer`.

#### Contentido

- [Instalación de class-transformer](#instalacion-class-transformer)
- [Configuración global de filtrado](#configuracion-global)
- [Filtrado de atributos](#filtrado-de-campos)

---

### Instalación de class-transformer {instalacion-class-transformer}

Cuando hablamos de los DTO en nuestros servicios instalamos el paquete `class-transformer`, pero si no viste ese artículo, el comando de instalación es el siguiente si usas `npm`:

```bash
npm install class-transformer
```

o si usas `yarn`:

```bash
yarn add class-transformer
```

Con esto ya tenemos nuestra única dependencia instalada.

### Configuración global de filtrado {#configuracion-global}

Ahora es importante realizar un ajuste en la configuración de nuestro proyecto para que las respuestas sean interceptadas y filtradas antes de salir de nuestro servidor, para ello vamos al archivo `main.ts` y nos aseguramos de tener agregada la siguiente configuración:

```typescript
import { Reflector } from '@nestjs/core';
import { ClassSerializerInterceptor, ValidationPipe } from '@nestjs/common';

async function bootstrap() {

  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,
      transformOptions: {
        enableImplicitConversion: true
      }
    })
  );
  app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));
  ...
}
bootstrap();

```

Observa que los cambios son referentes a `ClassSerializerInterceptor` y `enableImplicitConversion`.

### Filtrado de atributos {filtrado-de-campos}

Ahora para filtrar un campo específico de nuestras respuestas, solo debemos ir a la entidad cuyo atributo queramos filtrar y agregar el decorador `@Exclude()` del paquete `@ClassTansformer`.

Por ejemplo, para filtrar el campo `password` de nuestra entidad `User` haremos lo siguiente:

```typescript
import { Exclude } from 'class-transformer';

@Entity({ name: 'users' })
export class User {

  @Exclude()
  @Column({ type: 'varchar', length: 255 })
  password!: string;

  ...
}
```

Observa que tras la configuración y agregar este nuevo decorador en nuestra entidad, cuando creamos un nuevo usuario, la respuesta no contiene el atributo `password`, un ejemplo podamos verlo a continuación.

```json
{
  "id": 4,
  "email": "example-usar@example.com",
  "profile": {
    "id": 4,
    "name": "John",
    "lastName": "Doe",
    "avatarUrl": "https://myphotos.com/johndoe.jpg",
    "createdAt": "2026-09-06T14:07:02.154Z",
    "updatedAt": "2026-09-06T14:07:02.154Z"
  },
  "createdAt": "2026-09-06T14:07:02.154Z",
  "updatedAt": "2026-09-06T14:07:02.154Z"
}
```

Lo que hace este método más útil aún, es que el filtrado se aplica automáticamente a todos los servicios que acceden esta entidad de la base de datos.

Esto fue todo por ahora, te invito a ver el sigueinte artículo donde iniciamos con la autenticación de usuarios usando PassportJS, no te lo puedes perder.

---

Autor: Julio César Echeverri.
