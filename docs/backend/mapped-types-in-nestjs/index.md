---
layout: default
title: DTO y Mapped Types en NestJS
---

# DTO y Mapped Types en NestJS

En una entrada anterior estuvimos revisando asuntos referentes a _Data Transfer Objects_, cómo crearlos, usarlos y relacionarlos. Sin embargo, no revisamos escenarios donde un DTO describe de forma completa una entidad y funciona muy bien para usarse en un endpoint pero al mismo tiempo podría usarse en otro con algunas pequeñas variaciones.

Para este tipo de casos de uso contamos con una característica muy poderosa de NestJS llamada **Mapped Types**. En esta entrada vamos a aprender un poco sobre los más utilizados.

**Contenido**

- [Partial Type](#partial-type)
- [Pick Type](#pick-type)
- [Omit Type](#omit-type)

---

### Partial Type {#partial-type}

Cuando estamos construyendo DTOs para objetos que soportan patrones de creación/actualización, es conveniente tener un DTO de creación cuyos campos son requeridos y un DTO de actualización donde los mismos campos son opcionales.

Para evitar la replicación de código en este escenario NestJS nos ofrece el tipo **PartialType()** que convierte **todos** los campos del DTO que se pasa como argumento en campos opcionales.

Supongamos que tenemos el siguiente DTO de creación donde todos los campos son requeridos:

```typescript
import { ValidateNested } from "class-validator";
import { Type } from "class-transformer";

export class CreateUserDto {
  @IsEmail()
  @IsNotEmpty()
  @MinLength(8)
  email!: string;

  @IsString()
  @IsNotEmpty()
  password!: string;
}
```

Ahora, si deseamos crear el DTO para la operación de actualización del usuario, podríamos valernos de `PartialType()` de la siguiente forma:

```typescript
import { PartialType } from "nestjs/mapped-types";

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

De esta forma nuesto DTO de actualización aplicará las mismas reglas definidas para cada atributo del `CreateUserDto` pero hemos declarado que cada campo es opcional.

### Pick Type {#pick-type}

Con `PickType()` podemos construir un DTO (una clase) tomando solo algunos atributos de la clase base. Por ejemplo, si queremos crear un DTO que solo contenga el _email_ del usuario, la sintaxis podría ser:

```typescript
import { PickType } from "nestjs/mapped-types";

export class OnlyEmailDto extends PickType(CreateUserDto, ["email"] as const) {}
```

### Omit Type {#omit-type}

Con `OmitType()` podemos construir una nueva clase (un nuevo DTO) que tome todos los campos definidos en la clase base, omitiendo campos específicos que le indiquemos en los parámetros.
Si por ejemplo tenemos el siguiente DTO para la creación de un perfil:

```typescript
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

La sintaxis para crear un DTO que omita el _avatar_ pero conserve los demás campos, sería la siguiente:

```typescript
export class ProfileNoPhotoDto extends OmitType(CreateProfileDto, [
  "avatar",
] as const) {}
```

Los anteriores son los Mapped Types más utilizados aunque no los únicos disponibles, en la documentación oficial de NestJS puedes encontrar la [información sobre Mapped Types](https://docs.nestjs.com/techniques/validation#mapped-types) y te recomiendo darle una mirada.

Espero que esta entrada te haya resultado útil, nos vemos en la siguiente.

---

Autor: Julio César Echeverri M.
