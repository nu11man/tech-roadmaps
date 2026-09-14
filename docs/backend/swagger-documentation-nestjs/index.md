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

El primer paso que debemos ejecutar es la instalación de el paquete swagger en nuestro proyecto, si usamos `npm` ejecutamos la siguiente instrucción:

```bash
npm install --save @nestjs/swagger
```

En caso de usar `yarn` ejecutamos la siguiente acción:

```bash
yarn add @nestjs/swagger
```

### Configuración del proyecto {#ajuste-configuracion}

Una vez instalada el paquete `swagger` vamos a ir al archivo `main.ts` y agregamos la siguiente configuración:

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

En el fragmento anterior agregamos el módulo `SwaggerModule` a nuestra aplicación y al mismo tiempo creamos un documento de OpenAPI que puede ser accedido en la ruta `docs`. Adicionalmente, configuramos la ruta `swagger-json` para exponer la documentación generada por Swagger en formato JSON para que pueda ser consumido por modelos de IA o herramientas automatizadas.

### Documentación de DTO {#documentar-dto}

Para documentar los atributos de un DTO tenemos algunos decoradores muy útiles, como, por ejemplo:

- `@ApiProperty()`
- `@ApiPropertyOptional()`

Estos decoradores vienen del paquete `@nestjs/swagger`. Como el nombre indica uno se usa cuando la propiedad es requerida y el otro cuando es opcional.

Estos decoradores reciben un objeto de configuración que puede contener los siguientes atributos:

- `description`: Una breve descripción del atributo.
- `default`: Indica el valor por defecto.
- `minimum` o `maximum`: El valor mínimo o máximo que puede tomar.
- `type`: Indica explícitamente el tipo de dato. Si es un array `type: [tipo nativo]`.
- `maxLength` o `minLength`: Indica la longitud máxima y mínima.
- `enum`: Si el atributo es un enum, pasamos un array con los elementos.
- `example`: Permite indicar un ejemplo del contenido del atributo.
- `examples`: Permite indicar varios ejemplos para el atributo.

Entre otras posibles configuraciones, es posible profundizar más en estos decoradores en la [documentación oficial de NestJS para Swagger](https://docs.nestjs.com/openapi/types-and-parameters).

Para los DTO de tipo _Update_, aquellos que se derivan de DTOs definidos completamente a los que se aplica el mapped type `PartialType`, debemos mantenerlos como fueron definidos, pero ahora el paquete se debe importar desde `@nestjs/swagger` que se encarga de agregar el decorador `@ApiPropertyOptional()` internamente.

Un ejemplo de un _UpdateDTO_ se muestra a continuación:

```typescript
import { PartialType } from "@nestjs/swagger";
import { CreatePostDto } from "./create-post.dto";

export class UpdatePostDto extends PartialType(CreatePostDto) {}
```

### Documentación de entidades de base de datos {#documentacion-entidades}

### Documentación de endpoints {#documentar-endpoints}

Como ya sabemos los endpoints son gestionados desde los controladores, por lo tanto, vamos a ver los decoradores y parámetros que debemos usar en un controller con el fin de generar una buena documentación.

#### Descripción del endpoint {#descripcion-endpoint}

Para un endpoint podemos hacer uso del decorador `@ApiOperation()` y pasar los siguientes atributos en el objeto de configuración:

- `summary`: Un resumen de la operación realizada por el endpoint.
- `description`: Una descripción detallada de la operación realizada por el endpoint.
- `deprecated`: Una opción booleana que indica visualmente que el endpoint está deprecado.

Un ejemplo de los anterior es:

```typescript
 @ApiOperation({
    summary: 'Get all users',
    description: 'Retrieve a list of all users registered in the system',
    deprecated: true
  })
  @Get()
  getUsers() {
    return this.userService.getUsers();
  }
```

#### Describir query params {#describir-query-params}

#### Describir path params {#describir-path-params}

#### Descripción de las respuestas {#descripcion-respuestas}

Podemos describir posibles respuestas de nuestro endpoint mediante el decorador `@ApiResponse()` que puede recibir los siguiente atributos en su objeto de configuración:

- `status`: El valor numérico del código de respuesta HTTP.
- `description`: Una descripción de la respuesta entregada.
- `type`: Una clase decorada que indica los atributos de la respuesta.
- `isArray`: Un valor booleano que indica si la respuesta corresponde a un array.

Un ejemplo de la descripción de respuestas se muestra a continuación:

```typescript
export class UsersController {
  @Get()
  @ApiResponse({
    status: 200,
    description: "The users have been successfully retrieved.",
    type: UserDto,
    isArray: true,
  })
  findAll() {
    return [];
  }
}
```

También es importante mencionar que NestJS nos ofrece una gran lista de API responses predefinidas que nos pueden ahorrar mucho tiempo, una parte de la lista se muestra a continuación:

- `@ApiOkResponse()`
- `@ApiCreatedResponse()`
- `@ApiAcceptedResponse()`
- `@ApiNoContentResponse()`
- `@ApiMovedPermanentlyResponse()`
- `@ApiFoundResponse()`
- `@ApiBadRequestResponse()`
- `@ApiUnauthorizedResponse()`
- `@ApiNotFoundResponse()`
- `@ApiForbiddenResponse()`
- `@ApiMethodNotAllowedResponse()`
- `@ApiNotAcceptableResponse()`
- `@ApiRequestTimeoutResponse()`
- `@ApiConflictResponse()`
- `@ApiPreconditionFailedResponse()`
- `@ApiTooManyRequestsResponse()`
- `@ApiGoneResponse()`
- `@ApiPayloadTooLargeResponse()`
- `@ApiUnsupportedMediaTypeResponse()`
- `@ApiUnprocessableEntityResponse()`
- `@ApiInternalServerErrorResponse()`
- `@ApiNotImplementedResponse()`
- `@ApiBadGatewayResponse()`
- `@ApiServiceUnavailableResponse()`
- `@ApiGatewayTimeoutResponse()`
- `@ApiDefaultResponse()`

Un ejemplo que muestra el uso de los decoradores predefinidos se muestra a continuación:

```typescript
@Post()
@ApiCreatedResponse({ description: 'The record has been successfully created.'})
@ApiForbiddenResponse({ description: 'Forbidden.'})
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

### Seguridad de la documentación {#seguridad-documentacion}
