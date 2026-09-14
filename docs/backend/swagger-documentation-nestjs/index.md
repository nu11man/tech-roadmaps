---
layout: default
title: Documentación automática con Swagger en NestJS
---

# Documentación automática con Swagger en NestJS

#### Contenido

- [Instalación de dependencias](#instalacion-swagger)
- [Configuración del proyecto](#ajuste-configuracion)
- [Seguridad de la documentación](#seguridad-documentacion)
- [Documentación de DTO](#documentar-dto)
- [Documentación de entidades de base de datos](documentacion-entidades)
- [Documentación de endpoints](#documentar-endpoints)
  - [Descripción del endpoint](#descripcion-endpoint)
  - [Descripción de las respuestas](#descripcion-respuestas)
  - [Ejemplo de respuestas](#ejemplo-respuestas)

### Instalación de dependencias

```bash
npm install --save @nestjs/swagger
```

```bash
yarn add @nestjs/swagger
```

### Configuración del proyecto {#ajuste-configuracion}

```typescript
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('Blog API')
    .setDescription('The blog API description')
    .setVersion('1.0')
    .addTag('blog')
    .build();
  const documentFactory = () => SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('docs', app, documentFactory, {
    jsonDocumentUrl: '/swagger-json',
  });

  ...
}
await bootstrap();
```

### Documentación de DTO {#documentar-dto}

Para documentar los atributos de un DTO tenemos algunos decoradores muy útiles, como, por ejemplo:

- `@ApiProperty()`
- `@ApiPropertyOptional()`

Estos decoradores vienen del paquete `@nestjs/swagger`. Como el nombre indica uno se usa cuando la propiedad es requerida y el otro cuando es opcional.

Estos decoradores reciben un objeto de configuración que puede contener los siguientes atributos:

- `description`: Una breve descripción del atributo.
- `default`: Indica el valor por defecto.
- `minimum` | `maximum`: El valor mínimo o máximo que puede tomar.
- `type`: Indica explícitamente el tipo de dato. Si es un array `type: [tipo nativo]`.
- `maxLength` | `minLength`: Indica la longitud máxima y mínima.
- `enum`: Si el atributo es un enum, pasamos un array con los elementos.
- `example`: Permite indicar un ejemplo del contenido del atributo.
- `examples`: Permite indicar varios ejemplos para el atributo.

Entre otras posibles configuraciones, es posible profundizar más en estos decoradores en la [documentación oficial de NestJS para Swagger](https://docs.nestjs.com/openapi/types-and-parameters).

### Documentación de entidades de base de datos {#documentacion-entidades}

### Documentación de endpoints {#documentar-endpoints}

#### Descripción del endpoint {#descripcion-endpoint}

#### Descripción de las respuestas {#descripcion-respuestas}

### Seguridad de la documentación {#seguridad-documentacion}
