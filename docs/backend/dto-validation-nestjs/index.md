---
layout: default
title: DTO y Validación Automática de Datos en NestJS
---

# DTO y Validación Automática de Datos en NestJS

Para iniciar vamos a instalar dos depedencias muy importantes:

```bash
npm install --save class-validator class-transformer
```

O si estás usando _yarn_:

```bash
yarn add class-validator class-transformer
```

Ahora dentro del directorio del módulo que queremos validar, vamos a crear la carpeta _dto_ y dentro de esta vamos a crear los archivos que contienen las definiciones de los DTOs.

```typescript
import { IsEmail, IsNotEmpty, IsString } from "class-validator";

export class CreateUserDto {
  @IsEmail()
  @IsNotEmpty()
  email!: string;

  @IsString()
  @IsNotEmpty()
  password!: string;
}
```

**Nota**: Los DTO suelen estar asociados a acciones, por eso iniciamos con la acción `create` para el DTO asociado al método CREATE del controlador, los mismo aplica para el método UPDATE, etc.

**Nota**: Los decoradores de `class-validator` reciben objetos con algunas configuraciones, una de las más utilizada es la propiedad `message` que permite indicar un mensaje personalizado si hay un error de validación.

Para hacer uso de los DTOs que definimos, vamos al controlador de la ruta específica y en el handler aplicamos la siguiente sintaxis:

```typescript
import { Body, Post } from '@nestjs/common';
import { CreateUserDto } from './dto/user.dto';

@Post()
createUsers(@Body() body: CreateUserDto) {
  ...
}
```

En este handler, observamos como extraemos el cuerpo de la request (_body_) y lo almacenamos en el objeto `body`, que además obedece a la estructura y el contenido definido en el DTO `CreateUserDTO`.

Sin embargo, aunque hemos definido y utilizado nuestro primer DTO, aún debemos activarlo en la configuración de NestJS para que ejecute automáticamente las validaciones que encuentre a nivel global de clases que hagan uso de los decoradores de `class-validator`.

Activamos las validaciones yendo al archivo `main.ts` y agregando las siguientes líneas de `ValidationPipe`:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe())
  ...
}
bootstrap();
```

Si ahora vamos a nuestro cliente HTTP y enviamos una petición al endpoint `users` sin en contenido adecuado obtendremos un error, por otro lado, si enviamos una request cuyo contenido cumpla con las especificaciones descritas en el DTO, la ejecución se realizará exitosamente.
