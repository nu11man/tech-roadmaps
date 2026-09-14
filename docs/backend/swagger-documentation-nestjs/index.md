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

### Documentación de endpoints {#documentar-endpoints}

#### Descripción del endpoint {#descripcion-endpoint}

#### Descripción de las respuestas {#descripcion-respuestas}

### Seguridad de la documentación {#seguridad-documentacion}
