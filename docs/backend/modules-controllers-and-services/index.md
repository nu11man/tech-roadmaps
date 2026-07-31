---
layout: default
title: Módulos, Controladores y Servicios en NestJS
---

# Módulos, Controladores y Servicios en NestJS

Hasta este punto aprendimos a crear un proyecto NestJS y realizar algunas configuraciones básicas. En este artículo veremos tres conceptos fundamentales en el camino hacia el desarrollo de un servicio de backend, estos son: Módulo, Controladores y Servicios.

**Contenido**

- [Qué es un módulo](#que-es-un-modulo)
- [Cómo se crea un módulo](#crear-modulos)
- [Propiedades de los módulos](#propiedades-modulos)
- [Qué es un controlador](#que-es-un-controller)
- [Cómo se crea un controlador](#crear-controllers)
- [Propiedades de los controladores](#propiedades-controllers)
- [Qué es un servicio](#que-es-un-servicio)
- [Cómo se crea un servicio](#crear-servicios)

---

## Módulos en NestJS

### ¿Qué es un Módulo? {#que-es-un-modulo}

Un módulo es la unidad básica estructural que usa NestJS para organizar el código, agrupar componentes relacionados como **controladores** y **servicios**, y gestionar dependencias.

Cada aplicación tiene al menos un módulo raíz **AppModule** que funciona como el punto de entrada al servicio.

Podemos listar las características principales de un módulo en NestJS:

- Un módulo se define anotando una clase con el decorador `@Module`.

- Los módulos son _singletons_, es decir, mantienen una única instancia a lo largo del tiempo de vida del servicio. Además pueden ser importados por otros componentes sin complicaciones.

- El decorador `@Module` provee metadatos que NestJS utiliza para organizar la estructura de la aplicación.

### ¿Cómo se crea un Módulo? {#crear-modulos}

Una de las característica de NestJS CLI es que se puede user para implementar estructuras esquemáticas (y un Módulo es un esquemático definido por NestJS).

Una lista completa de los esquematicos se puede ver con el comando:

```bash
nest g --help
```

Se puede generar la instancia de un equemático a través de la siguiente sintaxis:

```bash
nest generate|g [options] <schematic> [name] [path]
```

Por ejemplo, para crear el módulo `user` nos ubicamos con la terminal en el directorio donde deseamos crear el módulo y ejecutamos el siguiente comando:

```bash
nest g module users
```

Como resultado, este comando crea un directorio para el módulo y sus archivos relacionados y, además actualiza el archivo `app.module.ts` inyectando el nuevo módulo de usuarios en el `AppModule`.

### ¿Qué propiedades podemos indicar en el módulo? {#propiedades-modulos}

Al decorador `@Module` le podemos pasar un objeto con 4 propiedades:

- `imports`: La lista de otros módulos que este módulo necesita para funcionar.

- `controllers`: Los controladores que manejarán las rutas asignadas a este módulo.

- `providers`: Los servicios, repositorios o clases que realizan la lógica de negocio y se pueden inyectar.

- `exports`: Los proveedores de este módulo que deseas compartir con otros módulos que lo importen.

---

## Controladores en NestJS

### ¿Qué es un Controlador? {#que-es-un-controller}

Un controlador o _Controller_ es una clase con la responsabilidad de gestionar las solicitudes (_requests_) que ingresan a la ruta controlada y a su vez de retornar las respuestas (_responses_) al cliente.

A los controladores se les asigna una ruta específica, correspondiente a un recurso, y éste se encarga de agrupar las rutas secundarias. Por ejemplo, para el recurso `tasks` tenemos la ruta principal `/tasks` y las rutas secundarias `/task/done`, `/tasks/pending`, etc.

Un _controller_ es una clase comentada con el decorador `@Controller('nombre-ruta')`, sus métodos se denominan _handlers_ y son los encargados de procesar la petición que corresponda con su ruta y método HTTP.

Los _handlers_ se crean como cualquier método de clase, pero comentado con un decorador que corresponde al verbo HTTP que deseamos manejar, estos decoradores reciben como parámetro la ruta secundaria a procesar.

Por lo tanto, un ejemplo de _controller_ puede ser el siguiente:

```typescript
import { Controller, Get, Post } from '@nestjs/common';

@Controller('users')
export class UsersController {

  @Get()
  getUsers() {
    return [{
      name: 'Julio',
      country: 'Colombia'
    }]
  }

  @Post('/admin')
  createAdminUser() {
    ...
  }
}
```

En el próximo artículo veremos como se parametrizan los _handlers_ para manejar rutas con _path params_ u obtener valores que se pasan como _query-params_. Por ahora, la implementación más básica nos basta para continuar.

Los controladores suelen sacar provecho de los Providers (que se pasan al módulo) a través del mecanismo de inyección de dependencias y lograr que la lógica de negocio y otras acciones sean manejadas por el proveedor, manteniendo el controlador limpio y con una única responsabilidad, direccionar las peticiones.

### ¿Cómo se crea un Controller? {#crear-controllers}

Al igual que con los módulos, NestJS nos ofrece un esquemático que podemos ejecutar desde la línea de comandos para crear la plantilla de un controlador.

```bash
nest g controller <resource-name>

# Ejemplo
nest generate controller users
```

Este esquemático creará la clase `UsersController` y actualizará automáticamente el Módulo del recurso `users` agregando el controlador en el array de `controllers`.

### Propiedades de los Controllers {#propiedades-controllers}

Los _controllers_ tienen un conjunto reducido de parámetros, podemos indicar unicamente un string con el nombre de la ruta que van a controlar o pasar un objeto con algunos atributos. Estos atributos son:

- `path`: Indica la ruta a controlar.
- `host`: Una string o expresión regular que restringe la ejecución de los _handlers_ para peticiones provenientes de un host o subdominio HTTP específico.
- `version`: Una string o array de strings utilizado para versionar los endpoints cuando el API Versioning está habilitado en la configuración del servicio.

Para habilitar el versionamiento de API debemos agregar el siguiente fragmento de código en el cuerpo de la función `bootstrap` en el archivo `main.ts`.

```typescript
async function bootstrap() {
  ...
  // Enable URI versioning globally (adds /v1/, /v2/, etc., to your paths)
  app.enableVersioning({
    type: VersioningType.URI,
  });
```

En este punto hay dos formas de aplicar el versionamiento, a nivel de controlador o a nivel de manejador.

A nivel de controlador tendríamos algo como lo siguiente:

```typescript
// Aplica el prefijo a todos los endpoints de la clase
@Controller({
  path: "users",
  version: "1", // Responde a: GET /v1/users
})
export class UsersV1Controller {
  @Get()
  findAllV1() {
    return "This action returns users from API Version 1";
  }
}
```

Por otra parte, a nivel de manejador hacemos uso del decorador `@Version`:

```typescript
import { Controller, Get, Version } from '@nestjs/common';

@Controller({
  path: 'customers',
})

export class CustomersController {

  // Responde a: GET /v2/customers
  @Version('2')
  @Get()
  getLegacyCustomers() {
    return 'Legacy customer data structure';
  }
```

Esta última estrategia resulta útil cuando quieres mantener múltiples versiones de los manejadores dentro del mismo controlador.

Podemos pasar un array de strings cuando queremos que un mismo manejador o controller responde a varias versiones.

```typescript
// Resuelve múltiples versiones:  GET /v3/customers y GET /v4/customers
  @Version(['3', '4'])
  @Get('experimental')
  getExperimentalCustomers() {
    return 'Beta version features for V3 and V4 clients';
  }
```

---

## Services y Providers en NestJS {#que-es-un-servicio}

Un provider puede ser un valor, una clase, una función Factory síncrona o asíncrona que está anotada con el decorator `@Injectable()` y se usan vía _Inyección de Dependencias_.

Tres de sus características son:

- Son _singletons_.
- Los _Providers_ deben inyectarse en un módulo para que sean utilizables en su interior.
- Se pueden exportar desde un módulo para que estén disponible en otro módulos que lo importen.

Y **¿qué es un _service_?** te estarás preguntando.

Un servicio es un concepto común en el desarrollo de software, en NestJS, es una clase anotada con el decorador `@Injectable()` igual que un _Provider_, la diferencia es que además de ser necesariamente una clase, también es la principal fuente de la lógica de negocio. Por ejemplo, será llamado por un controller para crear un elemento en una base de datos, validar datos entrantes, generar respuestas, etc.

**Nota**: Todo servicio es un _Provider_, pero no todos los _Providers_ son servicios.

### ¿Cómo se crea un Service? {#crear-servicios}

Como ya estarás acostumbrado, NestJS también ofrece un esquemático para crear servicios, la sintaxis es la siguiente:

```bash
nest g service <service-name>

# Ejemplo
nest generate service users
```

Con la línea anterior, el asistente de NestJS creará 2 archivos nuevos, la clase que define el servicio y su test unitario y, actualizará el atributo de `providers` del _Users Module_ definido en el archivo `users.module.ts`, para que dicho servicio pueda ser utilizado dentro de este módulo.

Finalmente el módulo de usuarios quedaría de esta forma:

```typescript
import { Module } from "@nestjs/common";
import { UsersController } from "./users.controller";
import { UsersService } from "./users.service";

@Module({
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

La clase del servicio quedaría como:

```typescript
import { Injectable } from "@nestjs/common";

@Injectable()
export class UsersService {
  sendHello() {
    return "hello";
  }
}
```

Y el controller que usaría los métodos de este servicio tendría la siguiente forma:

```typescript
import { Controller, Get, Post } from "@nestjs/common";
import { UsersService } from "./users.service";

@Controller("users")
export class UsersController {
  constructor(private userService: UsersService) {}

  @Get()
  getUsers() {
    return this.userService.sendHello();
  }
}
```

Es de suma importancia ver como se crea la instancia del servicio en el constructor del controlador, de esta forma se puede acceder al servicio y sus métodos con el operador `this`, como se ve en el _Handler_ **getUsers()**

Finalmente, a grandes rasgos estos son los que considero conceptos más fundamentales para trabajar con NestJS. Esto es todo por ahora respecto a Modules, Controllers y Providers, espero que te haya resultado útil esta información, nos vemos en los próximos artículos donde hablaremos de **DTO (data-transfer-objects**) y de acceso a query params, path params o body de las requests.

---

Autor: Julio César Echeverri.
