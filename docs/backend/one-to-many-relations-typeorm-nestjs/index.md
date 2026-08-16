---
layout: default
title: Relaciones Uno a Muchos en TypeORM y NestJS
---

# Relaciones Uno a Muchos en TypeORM y NestJS

Anteriormente vimos algunos artículos sobre configuración de TypeORM y creación de entidades básicas, luego revisamos las relaciones Uno a Uno entre entidades y en esta entrada vamos a pasar a revisar las relaciones Uno a Muchos entre nuestras entidades.

#### Contenido

- [Definición de las entidades](#definicion-de-entidades)
- [Registrar las Entidades en el Módulo](#inyeccion-entidades)
- [Acción de Creación](#create)
- [Acción de Actualización](#update)
- [Acción de Consulta](#read)
- [Acción de Eliminación](#delete)

---

### Definición de las entidades {#definicion-de-entidades}

De entradas anteriores veníamos trabajando con una entidad User y una entidad Perfil, ahora vamos a definir una nueva entidad llamada Posts, que hace referencia a los artículos del blog escritos por un usuario.

Un usuario tiene una relación one-to-one con un perfil, porque un usuario solo puede tener un perfil asociado. Sin embargo, un usuario puede escribir múltiples Posts (artículos) en el blog, por lo que esta relación será one-to-many, porque un usuario puede escribir varios Posts pero un post solo podrá ser creado por un único usuario.

Vamos a crear un nuevo recurso llamado posts en el directorio `src/`, creamos el módulo, el controller, servicios, DTOs, etc. En la carpeta `entities/` definiremos los campos habituales de la entidad como el ID, creation_date y update_date, luego algunos campos más específicos y finalmente definiremos la relación con la entidad usuario como se muestra a continuación:

```typescript
import {
  Entity,
  ManyToOne,
  JoinColumn
} from 'typeorm';
import { User } from '@src/users/entities/user.entity';

@Entity({ name: 'posts' })
export class Post {

  ...

  @ManyToOne(() => User, (user) => user.posts, { nullable: false })
  @JoinColumn({ name: 'user_id' })
  user!: User;
}
```

En el fragmento anterior indicamos que:

- `@ManyToOne()` indica que varias instancias de esta entidad estarán relacionadas con una instancia de la entidad Users.
- `nullable: false` indica que un post no se puede crear sin un usuario asociado.
- `(user) => user.posts` indica que existe una bidireccionalidad en la entidad User y que `post` está asociado al campo `posts` de la entidad User.
- `@JoinColumn()` indica que la relación se manejará con una clave foranea y que la columna tendrá el nombre `user_id`.

Ahora vamos a la entidad `User` y la modificaremos para crear la relación entre el usuario y los posts. Esto lo logramos con el siguiente fragmento.

```typescript
import {
  Column,
  CreateDateColumn,
  Entity,
  JoinColumn,
  OneToMany,
  OneToOne,
  PrimaryGeneratedColumn,
  UpdateDateColumn
} from 'typeorm';
import { Profile } from './profile.entity';
import { Post } from '@src/posts/entities/post.entity';

@Entity({ name: 'users' })
export class User {
  ...

  @OneToMany(() => Post, (post) => post.user)
  posts!: Post[];
}
```

En la entidad User hemos usado el decorador `@OneToMany` para indicar que el atributo `posts` contiene una referencia a _varios_ elemento de la entidad `Post`.

### Registrar las entidades en el módulo {#inyeccion-entidades}

Ahora debemos registrar en el módulo de usuarios `users.module.ts` el módulo de posts.

```typescript
import { Module } from "@nestjs/common";
import { UsersController } from "./users.controller";
import { UsersService } from "./users.service";
import { TypeOrmModule } from "@nestjs/typeorm";
import { User } from "./entities/user.entity";
import { Profile } from "./entities/profile.entity";

@Module({
  imports: [TypeOrmModule.forFeature([User, Profile, Posts])],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

### Acción de creación {#create}

### Acción de actualización {#update}

### Acción de consulta {#read}

### Acción de eliminación {#delete}

Espero que el contenido de esta entrada te haya resultado útil, nos vemos en el próximo post donde abordaremos el siguiente tipo de relación. La relación **Uno a Muchos**.

---

Autor: Julio César Echeverri M.
