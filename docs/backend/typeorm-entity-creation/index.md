---
layout: default
title: Creación de Entidades y Repository Pattern con TypeORM
---

# Creación de Entidades y Repository Pattern con TypeORM

En el artículo anterior vimos como instalar y configurar TypeORM en nuestro proyecto de NestJS. En esta oportunidad vamos a ver como crear una entidad de base de datos y acercarnos cada vez más al objetivo final, una aplicación de backend completamente funcional y lista para un ambiente productivo.

#### Contenido

- [Creación de la entidad](#creacion-de-entidades)
- [Columnas automáticas](#create-update-date-columns)
- [Repository Pattern](#repository-pattern)
- [Operación CREATE](#how-to-create)
- [Operación UPDATE](#how-to-update)
- [Operación DELETE](#how-to-delete)
- [Cómo consultar datos](#how-to-read)
  - [Obtener todos los registros de la tabla](#read-all)
  - [Obtener un registro por un campo específico](#read-by)
  - [Obtener registros que cumplan con condiciones](#read-by-condition)

---

### Creación de la entidad {#creacion-de-entidades}

En primer lugar, vamos a partir del supuesto de que tienes una ruta `/users`, entonces vamos al directorio `users` y creamos una carpeta llamada `entities` donde almacenaremos las definiciones de entidades de base de datos, estos archivos los nombramos siguiendo un estilo del tipo **user.entity.ts**.

En este archivo vamos a crear la clase que define a la entidad, la definición de una entidad muy básica tiene el siguiente contenido:

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

@Entity({ name: "users" })
export class User {
  @PrimaryGeneratedColumn()
  id!: number;

  @Column({ type: "varchar", length: 255, unique: true })
  email!: string;

  @Column({ type: "varchar", length: 255 })
  password!: string;
}
```

Del código anterior cabe destacar que todos los decoradores vienen del paquete `typeorm`.
Adicionalmente, de los decoradores utilizados podemos decir que:

- **`Entity()`**: Es el decorador que define la entidad que corresponde a una tabla de la base de datos. Normalmente los nombres de las tablas se escriben en plural y con \_ cuando corresponde a nombres compuestos. Este decorador recibe un objeto como parámetro que describe (entre otras opciones) el nombre de la tabla.

- **`Column()`**: Este decorador describe un atributo del objeto que se corresponderá con una columna de la tabla. Este decorador describe un objeto con varios atributos, entre ellos, el tipo de dato, la longitud (si aplica), si es único o no, si es puede ser nulo, etc.

- **`PrimaryGeneratedColumn()`**: Este decorador nos permite indicar que un atributo corresponde a la llave primaria de la tabla, es decir, el identificador. Este decorador maneja la lógica para la gestión de la _primary key_ de forma automática. Por defecto, el tipo de ID es un valor numérico autoincremental, pero puede indicarse también otra estratega como `uuid` entre otros.

### Columnas automáticas {#create-update-date-columns}

Una de las prácticas más importantes en la gestión de entidades de base de datos es el registro de fechas de creación y última actualización de cada elemento de la tabla. Dado que este es un patrón muy frecuente en las aplicaciones de backend, contamos con dos decoradores muy útiles:

- **`CreateDateColumn()`**: Este decorador registra automáticamente el _timestamp_ de creación de la entidad en la tabla.
- **`UpdateDateColumn()`**: Este decorador actualiza automáticamente el _timestamp_ cada vez que se actualiza el registo de la entidad en la tabla.

Ambos decoradores reciben un objeto de configuración, mi recomendación es, como mínimo configurar los siguientes aspectos que se muestran en el ejemplo:

```typescript
import {
  CreateDateColumn,
  UpdateDateColumn
} from 'typeorm';

@Entity({ name: 'users' })
export class User {

  ...

  @CreateDateColumn({
    name: 'created_at',
    type: 'timestamptz',
    default: () => 'CURRENT_TIMESTAMP'
  })
  createdAt!: Date;

  @UpdateDateColumn({
    name: 'updated_at',documentación oficial de NestJS y TypeORM repository pattern
    type: 'timestamptz',
    default: () => 'CURRENT_TIMESTAMP'
  })
  updatedAt!: Date;
}
```

### Repository Pattern {#repository-pattern}

Dado que TypeORM nos permite hacer el mapping entre los objetos y sus relaciones, también nos ofrece la posibilidad de trabajar con el patrón de diseño **Repository**, es decir que cada entidad cuenta con los métodos para realizar múltiples acciones comunes de cara a la base de datos. Más información puede encontrarse la [documentación oficial de NestJS y TypeORM repository pattern](https://docs.nestjs.com/techniques/database#repository-pattern).

El uso del Repository Pattern con TypeORM consiste realmente en crear una entidad (cómo lo hicimos anteriormente) e inyectarla en el servicio que hace uso de ella. Por ejemplo, dada entidad `User` que creamos recientemente, vamos a ir al servicio de usuarios `users.service.ts` y en el constructor inyectaremos un Repository instanciado especialmente para esta entidad. Veamos el siguiente código:

```typescript
import { Injectable } from '@nestjs/common';
import { Repository } from 'typeorm';
import { InjectRepository } from '@nestjs/typeorm';
import { User } from './entities/user.entity';

@Injectable()
export class UsersService {

  constructor(
    @InjectRepository(User) private userRepository: Repository<User>
  ) {}

```

En este código hemos realizado tres acciones fundamentales:

1. Utilizamos el decorador `@InjectRepository` para crear una instancia de un repositorio de la entidad `User` (que pasamos como argumento).
2. Declaramos el nombre de la variable a la que asignaremos el resporitorio y a través de la cual realizaremos las acciones en nuestro servicio.
3. Definimos el tipo de la variables `userRepository` con `Repository<User>`.

Una vez inyectado el Repository en nuestra clase de servicio, podemos comenzar a hacer uso de las operaciones disponibles en eĺ, como podemos ver a modo de ejemplo en el siguiente código:

```typescript
@Injectable()
export class UsersService {

  ...

  async findUserById(id: number) {
    const user = await this.userRepository.findOneBy({ id });

    if ( !user )
      throw new NotFoundException(`User with id ${id} not found`);

    return user;
  }

```

En el _repository_ contamos con una basta cantidad de métodos que merecen la pena ser estudiados, en las siguientes secciones veremos las operaciones más básicas.

### Operación CREATE {#how-to-save}

Para guardar el contenido de una entidad en la base de datos contamos con el método `.save()` del repositorio, al cual solo basta con pasarle un objeto que cumpla con la estructura de la entidad.

```typescript
async create(body: CreateUserDto) {
  const newUser = await this.userRepository.save(body);
  return newUser;
}
```

### Operación UPDATE {#how-to-update}

Para hacer la actualización de una entidad debemos recuperar el contenido actual de la entidad y luego hacer un `.merge()` del objeto existente y del objeto que contiene los cambios, para posteriormente guardar el nuevo objeto.

Veamos un ejemplo de lo descrito:

```typescript
async updateUser(id: number, changes: UpdateUserDto) {
  const user = await this.userRepository.findOne(id);
  const updatedUser = await this.userRepository.merge(user, changes);
  const savedUser = await this.userRepository.save(updatedUser);

  return savedUser;
}
```

### Operación DELETE {#how-to-delete}

Hemos visto como guardar y cómo actualizar un registro de la tabla, eliminar un elemento también se puede considerar trivial ya que contamos con un método específico para ello:

```typescript
async deleteUser(id: number) {
  await this.usersRepository.delete(id);
  return { message: "User deleted" };
}
```

### Cómo consultar datos {#how-to-read}

Hasta este punto vimos como _almacenar_, _actualizar_ y _eliminar_ registros en las tablas de la base de datos. En esta sección vamos a ver como leer un registro de la base de datos utilizando el _repository_.

#### Obtener todos los registros de la tabla {#read-all}

Podemos obtener todos los registros de la tabla (con las implicaciones de performance que esto trae) haciendo uso del método `find()`. Veamos un ejemplo donde obtenemos todos los usuarios.

```typescript
async getAllUsers() {
  const users = await this.usersRepository.find();
  return users;
}
```

#### Obtener un registro por un campo específico {#read-by}

La mayoría de la veces queremos obtener el registro correspondiente a un ID (primary key) específica, por ejemplo, para obtener un usuario dado su ID:

```typescript
async getUserById(id: number) {
  const user = await this.usersRepository.findOneBy({ id });

  if (!user) {
    throw new NotFoundExecption(`User with id ${id} not found`);
  }
  return user;
}
```

#### Obtener registros que cumplan con condiciones {#read-by-conditions}

En otras ocasiones deseamos traer una lista de registros que satisfacen una clausula, es posible lograr esto a través del atributo `where` en el objeto que se pasa como argumento del método.

```typescript
async getUserByEmail(email: string) {
    const user = await this.userRepository.findOne({
      where: { email },
    });
    return user;
  }
```

Cómo hemos observado de los ejemplos anteriores, obtener los registros de la base de datos y operar con ellos como objetos de Javascript es la mágia de TypeORM (en general es la promesa de los ORMs). Puedes consultar [la documentación oficial de **TypeORM**](https://typeorm.io/docs/working-with-entity-manager/find-options) aquí para mayor información sobre las operaciones disponibles.

**Nota**: Observa que en este artículo solo hemos querido introducir los métodos que podemos utilizar pero es importante que tus implementaciones sean más robustas que los ejemplos aquí presentados. Es decir, debería tener un manejo adecuado de excepciones, mayor validación de tipos, etc.

---

Espero que este artículo te haya resultado útil, nos vemos en el próximo para conversar un poco sobre las relaciones One-to-One con TypeORM.

Autor: Julio César Echeverri M.
