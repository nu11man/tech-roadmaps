---
layout: default
title: Hooks de TypeORM en NestJS
---

# Hooks de TypeORM en NestJS

En múltiples ocasiones nos podemos ver la necesidad de realizar operaciones sobre un atributo de una entidad antes de guardar la información en la base de datos o justo después de extraerla. A esos momentos por los que transcurre la transacción le llamamos el _ciclo de vida_ de la transacción y las funciones que realizan esas acciones les llamamos _hooks_

En esta entrada veremos un caso de uso muy frecuente en las bases de datos y es el _hashing de contraseñas_, en otras palabras, la acción de cifrar una contraseña antes de guardarla.

#### Contentido

- [Instalación de BCrypt](#instalacion-bcrypt)
- [Agregar Hooks en TypeORM](#hooks-en-typeorm)
- [El Hook @BeforeInsert y hashing de password](#before-insert-hook)
- [Verificación de password](#verificacion-de-contrasena)

---

### Instalación de BCrypt {#instalacion-bcrypt}

Una de los paquetes más utilizados en el entorno de NodeJS para cifrar/descifrar información es **BCrypt** este paquete por defecto está construido con Javascript, para tener soporte de tipos con TypeScript, debemos instalar un paquete adicional (en modo desarrollo).

```bash
npm install bcrypt

npm install --dev @types/bcrypt
```

O si estás usando `yarn` ejecutamos las siguientes líneas:

```bash
yarn add bcrypt

yarn add --dev @types/bcrypt
```

### Agregar Hooks en TypeORM {#hooks-en-typeorm}

En TypeORM contamos con una variada lista de hooks (también llamados `listeners`), funciones que se ejecutan en momentos específicos de una transacción, algunos de estos hooks son:

- @AfterLoad
- @BeforeInsert
- @AfterInsert
- @BeforeUpdate
- @AfterUpdate
- @BeforeRemove
- @AfterRemove
- @BeforeSoftRemove
- @AfterSoftRemove
- @BeforeRecover
- @AfterRecover

Puedes ver la [documentación oficial](https://typeorm.io/docs/listeners-and-subscribers) para explorar cada uno de ellos.

### El Hook @BeforeInsert en TypeORM {#before-insert-hook}

En TypeORM podemos agregar un hook yendo a la entidad específica donde queremos realizar la acción, por ejemplo, podemos agregar una función para aplicar un `hashing` en la contraseña del usuario. Para ello vamos a la entidad `User` y agregamos el siguiente fragmento de código:

```typescript
import { BeforeInsert } from 'typeorm';
import * as bcrypt from 'bcrypt';

@Entity({ name: 'users' })
export class User {
  ...

  @BeforeInsert()
  async hashPassword(): Promise<void> {
    this.password = await bcrypt.hash(this.password, 10);
  }
}
```

Al marcar cualquier método con uno de los decoradores tipo _hook_/_listener_ que listamos anteriormente, haremos que TypeORM ejecute dicho método en el momento preciso. En este caso `@BeforeInsert` se ejecuta antes de hacer un `save` de la entidad, pero debemos tener en cuenta que previamente debemos crear el objeto en memoria usando el método `.create()`, por lo que nuestro servicio de creación de usuarios deberíamos ajustarlo como se ve a continuación:

```typescript
async createUser(user: CreateUserDto) {
  try {
    const newUser = this.userRepository.create(user);
    const createdUser = await this.userRepository.save(newUser);
    return createdUser;
  } catch (error) {
    throw new BadRequestException('Error creating user');
  }
}
```

El resultado de ejecutar el método de creación de un usuario en el servicio de `Users` nos entregará la siguiente respuesta:

```json
{
  "id": 4,
  "email": "example-usar@example.com",
  "password": "$2b$10$FrkdGRfWN4usWx6zMW66ouPwsYsKeDwhVfaUCnAZc5QiMXwrH7gKi",
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

Observa que el campo `password` ahora no corresponde a la contraseña en texto plano sino a un hash criptográfico.

Adicionalmente te invito a ver nuestra otra entrada sobre el filtrado de campos en las respuestas de nuestros servicios, como puedes ver, no es útil que retornemos el atributo password en las consultas de nuestros usuarios.

### Verificación de password {#verificacion-de-contrasena}

Ahora para cerrar el ciclo vamos ver como comparar una contraseña entregada por el usuario y la contraseña _hasheada_ que tenems almacenada en la base de datos.
