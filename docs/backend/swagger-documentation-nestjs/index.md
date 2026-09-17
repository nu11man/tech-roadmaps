---
layout: default
title: Documentación automática con Swagger en NestJS
---

# Documentación automática con Swagger en NestJS

Cuando estamos construyendo un servicio web, una documentación de alta calidad es tan importante como la calidad del código que escribimos. Finalmente la documentación terminamos consumiendo nosotros mismos en el futuro o los desarrolladores de otros servicios o clientes que vayan a consumir nuestros recursos.

En esta entrada vamos a ver todo lo que necesitamos para generar automáticamente una documentación de alta calidad con **Swagger**, formalmente conocido ahora como la especificación **OpenAPI**.

#### Contenido

- [Instalación de dependencias](#instalacion-swagger)
- [Configuración del proyecto](#ajuste-configuracion)
- [Seguridad de la documentación](#seguridad-documentacion)
- [Documentación de DTO](#documentar-dto)
- [Documentación de endpoints](#documentar-endpoints)
  - [Descripción del endpoint](#descripcion-endpoint)
  - [Documentar path params](#documentar-path-params)
  - [Documentar query params](#documentar-query-params)
  - [Documentar headers](#documentar-headers)
  - [Documentar rutas protegidas con Bearer Token](#documentar-rutas-protegidas)
  - [Descripción de las respuestas](#descripcion-respuestas)

---

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
- `type`: Indica explícitamente el tipo de dato. Si es un array `type: [tipo_dato]`.
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

#### Describir path params {#documentar-path-params}

Como vimos en una entrada anterior dedicada a los llamados _query params_ y _path params_, aprendimos que los _path params_ corresponden a segmentos variables o parametrizables de una ruta, son valores variables normalmente utilizados para identificar un recurso específico dentro de un conjunto.

Documentar los _path params_ es de suma importancia para mejorar la experiencia de desarrollo de quienes consumen nuestra API. Podemos documentar un _path param_ utilizando el decorador `@ApiParam()`. Observemos el siguiente ejemplo:

```typescript
@Controller('products')
export class ProductsController {

@Get(':id')
@ApiParam({
  name: 'id',
  type: String,
  description: 'The unique identifier of the product',
  example: 'prod_95x82103',
})
findOne(@Param('id') id: string) {
  return `This action returns product #${id}`;
}
```

#### Describir query params {#documentar-query-params}

Los _query params_ corresponden a valores que se pasan en la URL después del símbolo `?` y que vienen dados en forma de `key=value` separados por un símbolo `&`. Los _query params_ son el mecanismo utilizado para filtrar, agrupar u ordenar recursos en un conjunto.

Un ejemplo típico de una URL con _query params_ puede ser un endpoint retorna una lista de vehículos pero puede filtrar por marca y además tiene paginación y retorna máximo 5 elementos:

```typescript
https://example.com/cars?brand=chevrolet&limit=5
```

Podemos documentar los _query params_ de dos formas, la primera es individualmente, y está bien cuando son uno o dos query params máximo, como vemos a continuación:

```typescript
@Get('/cars')
@ApiOperation({ summary: 'Return a list of cars with pagination' })
@ApiQuery({
  name: 'limit',
  type: Number,
  required: false,
  description: 'Number of items to return per page',
  example: 10,
})
@ApiQuery({
  name: 'brand',
  type: String,
  required: false,
  description: 'Filter cars by brand name',
})
findAll(@Query('limit') limit?: number, @Query('brand') brand?: string) {
  return `Returns cars filtered by "${brand}", limited to ${limit} items.`;
}
```

Por otra parte, cuando tenemos varios _query params_ que pueden entrar en nuestro servicio, lo más adecuado es crear un DTO, es decir, una clase que en sus atributos describe cada uno de los _query params_ y además los anota con los decoradores `@ApiProperty()` y `@ApiPropertyOptional()` como vemos en el siguiente ejemplo:

```typescript
// products-query.dto.ts
import { ApiPropertyOptional } from "@nestjs/swagger";

export class ProductsQueryDto {
  @ApiPropertyOptional({
    description: "Number of items to return",
    example: 10,
  })
  limit?: number;

  @ApiPropertyOptional({ description: "Filter cars by brand name" })
  brand?: string;
}
```

Luego al hacer uso del DTO que definimos para los _query params_, Swagger automáticamente documentará cada uno de ellos, nuestro controlador quedará más limpio como se ve en el siguiente fragmento:

```typescript
@Get('/cars')
findAll(@Query() query: ProductsQueryDto) {
// Swagger automaticament documenta 'limit' y 'brand'
}
```

#### Documentar headers {#documentar-headers}

En NestJS podemos documentar un header utilizando el decorador `@ApiHeader()` del paquete `@nestjs/swagger`.

El decorador `@ApiHeader()` recibe un objeto con los siguientes atributos:

- `name`: El nombre del header,
- `description`: La descripción del header.
- `required`: Un valor booleano que indica si el header es obligatorio u opcional.
- `example`: Un valor que se muestra a modo de ejemplo.
- `schema`: Un objeto que posee los atributosThrottle `type` y opcionalmente `example`.

Veamos un ejemplo:

```typescript
@ApiHeader({
  name: 'x-correlation-id',
  description: 'Unique identifier used to trace the request',
  required: true,
  schema: {
    type: 'string',
    example: '558e0488-e29b-41d4-a716-446655440000'
  }
})
```

**Nota**: Si deseamos documentar varios headers en el mismo endpoint, solo tenemos que agregar múltiples decoradores `@ApiHeader()`

Por otra parte, si tenemos un header que se usa múltiples veces en diferentes lugares, podemos optar por generar una documentación reusable de headers creando nuestro propio decorador personalizado.

```typescript
// api-correlation-id.decorator.ts

import { applyDecorators } from "@nestjs/common";
import { ApiHeader } from "@nestjs/swagger";

export function ApiCorrelationId() {
  return applyDecorators(
    ApiHeader({
      name: "x-correlation-id",
      description: "Unique identifier used to trace the request.",
      required: true,
      schema: {
        type: "string",
        example: "550e8400-e29b-41d4-a716-446655440000",
      },
    }),
  );
}
```

Luego en el controlador simplemente lo importamos y lo usamos de la siguiente manera:

```typescript
import { ApiCorrelationId } from '@decorators/api-correlation-id.decorator';

@Get()
@ApiCorrelationId()
getUsers() {
  ...
}
```

En caso de que tengas múltiples implementaciones de decoradores personalizados, una estructura como la siguiente puede ser recomendable:

```
src/
├── common/
│   ├── decorators/
│   │  ├── swagger/
|   |  │   ├── api-correlation-id.decorator.ts
|   |  │   ├── api-client-version.decorator.ts
|   |  │   ├── api-device-id.decorator.ts
|   |  │   └── index.ts
|   |  ├── auth/
|   |  │   └── ...
|   |  └── index.ts
│   ├── guards/
│   ├── interceptors/
│   ├── filters/
│   └── pipes/
│
├── modules/
│   ├── users/
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   └── ...
│   └── auth/
│       └── ...
│
├── app.module.ts
└── main.ts
```

Observa que dentro del directorio `swagger` tenemos un archivo `index.ts` que agrupa y exporta todos los decoradores, con una sintaxis como la siguiente:

```typescript
// src/common/decorators/swagger/index.ts

export * from "./api-correlation-id.decorator";
export * from "./api-client-version.decorator";
export * from "./api-device-id.decorator";
```

Por lo que al momento de importarlos la sintaxis será la siguiente:

```typescript
import {
  ApiCorrelationId,
  ApiClientVersion,
  ApiDeviceId,
} from "@/common/decorators/swagger";
```

Una nota final a considerar es que si el header solo se reutiliza unicamente dentro de los endpoints expuestos en un controller y no en los demás, deberíamos entonces ubicar los decoradores personalizados, dentro del módulo que hace uso de el, de este modo el directorio `common` se mantendrá más limpio y unicamente tendrá implementaciones realmente compartidas.

#### Documentar rutas protegidas con Bearer Token {#documentar-rutas-protegidas}

La autorización es un header más. Sin embargo, no solemos documentar el header de autorización que usa un Bearer Token usando el decorador `@ApiHeader()`, sino uno de los decoradores dedicados de `@nestjs/swagger`. El decorador `ApiBearerAuth()`.

```typescript
import { ApiBearerAuth } from '@nestjs/swagger';

@Get()
@ApiBearerAuth()
getUsers() {
  ...
}
```

Para que la documentación describa correctamente el esquema de autorización con Bearer Token y agregue automáticamente el botón de _Authorize_. Debemos agregar el método `.addBearerAuth()` en la configuración de Swagger en el archivo `main.ts`.

```typescript
const config = new DocumentBuilder()
  .setTitle("My API")
  .setDescription("API documentation")
  .setVersion("1.0")
  .addBearerAuth()
  .build();
```

#### Descripción de las respuestas {#descripcion-respuestas}

Podemos describir posibles respuestas de nuestro endpoint mediante el decorador `@ApiResponse()` que puede recibir los siguiente atributos en su objeto de configuración:

- `status`: El valor numérico del código de respuesta HTTP.
- `description`: Una descripción de la respuesta entregada.
- `type`: Una clase decorada que indica los atributos de la respuesta (algunas veces se corresponde con una clase decorada de una entidad de base de datos).
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

**Nota**: Si queremos que nuestra documentación muestre ejemplos de respuesta de nuestro endpoint, debemos asegurarnos de pasar una clase en el atributo `type` del objeto de configuración que se pasa a `@ApiResponse`. Esta clase debe describir la forma de la respuesta, pensemos en un DTO de respuesta. Es una clase comentada con los decoradores `@ApiProperty()` y `@ApiPropertyOptional()`.

### Seguridad de la documentación {#seguridad-documentacion}

El mejor esquema para proteger la documentación de la aplicación es no activarla en el ambiente productivo, solamente en `development` y `staging`, y además, ponerla detrás de una VPN. Sin embargo, si nuestro proyecto no es aún tan grande como para levantar una estructura de ambientes, VPN y credenciales, sino que, estamos hablando de un proyecto piloto o una prueba de concepto, podemos valernos de un esquema básico de autenticación.

Para aplicaciones de NestJS que usan Express (lo que ocurre por defecto), de hecho, NestJS usa el paquete `swagger-ui-express` para la documentación de Swagger. Nosotros podemos agregar el paquete `express-basic-auth` que podemos instalar con la siguiente línea:

```bash
npm install express-basic-auth
```

o usando yarn:

```bash
yarn add express-basic-auth
```

Luego debemos ajustar la configuración en el archivo `main.ts`, allí vamos a agregar un middleware que se encargará de verificar si se está haciendo una request a las rutas `/docs` o `/docs-json` (que son nuestras rutas de documentación) y si es el caso, entonces solicitará un usuario y contraseña para poder liberar la documentación.

La configuración unicamente varia al agregar el middleware en la aplicación, veamos el ejemplo:

```typescript
async function bootstrap() {
// Protect Swagger UI and Swagger JSON
  const swaggerUsername = configService.getOrThrow<string>('swagger.username');
  const swaggerPassword = configService.getOrThrow<string>('swagger.password');

  app.use(
    ['/docs', '/docs-json'],
    expressBasicAuth({
      challenge: true,
      users: {
        [swaggerUsername]: swaggerPassword
      }
    })
  );

  // Setups and build Swagger documentation
  const config = new DocumentBuilder()
    .setTitle('Blog API')
    .setDescription('The blog API description')
    .setVersion('1.0')
    .addTag('blog')
    .build();

  ...
}
```

Recuerda que como estamos usando el `ConfigService` debemos actualizar con las nuevas variables el archivo de ambiente `.env`, el schema validator y el loader, tal cómo vimos en el post sobre variables de entorno.

Ahora si levantas el servidor y vas a la ruta de documentación encontrarás que solicita las credenciales antes de poder visualizarlas. Recuerda elegir una contraseña con un buen nivel de complejidad. Observa que, aunque ahora solicitamos credenciales, no estamos protegidos ante ataques de fuerza bruta, es decir que un atacante podría enviar miles de peticiones tratando de adivinar nuestras credenciales. Este tópico no solo es importante sino que lo podemos mitigar protegiendo algunas rutas como el `/loging` usando el decorador `@Throttle()` del paquete `@nestjs/throttle` sino también configurando adecuadamente los _rate limiters_ de Nginx. Todo esto lo veremos en futuros posts.

Eso fue todo por hoy, en esta entrada pudimos ver todos los mecanismos que podemos utilizar para generar una documentación de alta calidad para nuestra aplicación, vimos además como protegerla usando un esquema básico de usuario y contraseña para que la estructura de nuestro servicio no esté abierta públicamente en internet. Con esto finalizamos el apartado de documentación y nos vamos a mover para ver los pasos de configuración de nuestro proyecto de cara al despliegue en un ambiente de producción.

---

Autor: Julio César Echeverri Marulanda
