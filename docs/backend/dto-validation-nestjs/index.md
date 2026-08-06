---
layout: default
title: DTO y Validación Automática de Datos en NestJS
---

# DTO y Validación Automática de Datos en NestJS

**Contenido**

- [Instalación de dependencias](#instalacion-dependencias)
- [Validación de objetos simples](#validacion-objetos-simples)
- [Validación de objetos anidados](#validacion-objetos-anidados)
- [Uso de DTO en el Controlador](#usar-dto-en-el-controlador)

---

### Instalación de dependencias {#instalacion-dependencias}

Para iniciar vamos a instalar dos depedencias muy importantes:

```bash
npm install --save class-validator class-transformer
```

O si estás usando _yarn_:

```bash
yarn add class-validator class-transformer
```

### Validación de objetos simples {#validacion-objetos-simples}

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

**Nota**: Los DTO suelen estar asociados a acciones, por eso iniciamos con la acción `create` para el DTO asociado al método CREATE del controlador, lo mismo aplica para el método UPDATE, etc.

**Nota**: Los decoradores de `class-validator` reciben objetos con algunas configuraciones, una de las más utilizada es la propiedad `message` que permite indicar un mensaje personalizado si hay un error de validación.

### Validación de objetos anidados {#validacion-objetos-anidados}

En la mayoría de ocasiones, suele ocurrir que manejamos DTOs donde una o más propiedades corresponden a objetos anidados, por ejemplo un objeto con la siguiente forma:

```json
  "email": "usuario@ejemplo.com",
  "password": "asdasd",
  "profile": {
    "name": "Juan Carlos",
    "lastName": "Molina",
    "avatar": null,
    "phone": "3247876523"
  }
```

En este caso vamos a tener definido el DTO `CreateUserDto`, pero adicionalmente crearemos un DTO que describa el objeto _Profile_, lo llamaremos `CreateProfileDto`.

```typescript
import { IsUrl, IsNotEmpty, IsString, IsOptional } from "class-validator";

export class CreateProfileDto {
  @IsString()
  @IsNotEmpty()
  name!: string;

  @IsString()
  @IsNotEmpty()
  lastName!: string;

  @IsUrl()
  @IsOptional()
  avatar!: string;

  @IsString()
  @IsNotEmpty()
  phone!: string;
}
```

Ahora para actualizar el `CreateUserDto` e incluir el atributo `profile` como un DTO anidado y, que además garanticemos las validaciones internas del mismo, realizamos el siguiente ajuste en el DTO de creación de usuarios.

### Uso de DTO en el Controlador {#usar-dto-en-el-controlador}

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
