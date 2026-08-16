---
layout: default
title: Relaciones Uno a Muchos en TypeORM y NestJS
---

# Relaciones Uno a Muchos en TypeORM y NestJS

Anteriormente vimos algunos artículos sobre configuración de TypeORM y creación de entidades básicas, luego revisamos las relaciones Uno a Uno entre entidades y en esta entrada vamos a pasar a revisar las relaciones Uno a Muchos entre nuestras entidades.

#### Contenido

- [Definición de las entidades](#definicion-de-entidades)
- [Registrar las Entidades en el Módulo](#inyeccion-entidades)
- [Repository Pattern](#repository-pattern)
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

Ahora debemos registrar en el módulo de `Posts` la entidad `Post` que definimos recientemente.

```typescript
import { Module } from "@nestjs/common";
import { PostsService } from "./posts.service";
import { PostsController } from "./posts.controller";
import { TypeOrmModule } from "@nestjs/typeorm";
import { Post } from "./entities/post.entity";

@Module({
  imports: [TypeOrmModule.forFeature([Post])],
  controllers: [PostsController],
  providers: [PostsService],
  exports: [PostsService],
})
export class PostsModule {}
```

Si ahora ejecutamos el servidor, encontraremos la nueva tabla `Posts` en nuestra base de datos, debemos aclarar aquí que la relación a nivel de esquema unicamente la tiene registrada la entidad `Post`, es decir, no hay una referencia actualmente desde Users hacia Posts porque son los Posts los que apuntan al usuario y no al revés, _es la entidad más débil la que carga la relación_.

### Repository Pattern {#repository-pattern}

Antes de realizar las acciones debemos recordar que utilizamos el _Repository Pattern_, por lo que nuestro servicio de Posts y su constructor deberan tener la siguiente forma:

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { Repository } from 'typeorm';
import { InjectRepository } from '@nestjs/typeorm';
import { CreatePostDto } from './dto/create-post.dto';
import { UpdatePostDto } from './dto/update-post.dto';
import { Post } from './entities/post.entity';

@Injectable()
export class PostsService {
  constructor(
    @InjectRepository(Post)
    private readonly postRepository: Repository<Post>
  ) {}

  ...

}
```

Por otra parte, no hay cambios muy relevantes respecto a las acciones que podemos realizar con estas entidades ahora que tenemos la relación _uno a muchos_, veamos a continuación.

### Acción de creación {#create}

Dado que, a diferencia de las entidades más simples que creamos antes, la entidad `Post` ahora requiere una referencia a un usuario asociado con el post, debemos tener el cuidado de pasar esta referencia como se espera en la defición de la entidad.

En este caso, la entidad espera un atributo `user` que recibe un objeto de tipo `User` entity, no necesariamente completo, pero como mínimo debe contener el ID de dicho usuario. El método `create` quedará entonces de la siguiente forma:

```typescript
async create(post: CreatePostDto) {
  const newPost = await this.postRepository.save({
    ...post,
    user: { id: post.userId }
  });
  const createdPost = await this.findOne(newPost.id);
  return createdPost;
}
```

Aquí usamos además el método `.findOne()` que definiremos más adelante y que nos permitirá retornar el contenido del post creado y del usuario y perfil asociados a este nuevo post.

### Acción de actualización {#update}

Para realizar la actualización de un post, obtenemos el post original a través de su ID utilizando nuestro método personalizado `.findOne()`, posteriormente hacemos un `merge` entre el contenido actual y el nuevo. Finalmente hacemos un `.save()` con la nueva información.

```typescript
async update(id: number, updatePostDto: UpdatePostDto) {
  const post = await this.findOne(id);
  const updatedPost = this.postRepository.merge(post, updatePostDto);
  const newPost = await this.postRepository.save(updatedPost);

  return newPost;
}
```

### Acción de consulta {#read}

En cuanto a la consulta de datos el mismo patrón se mantiene, vamos a crear un método personalizado `.findOne()` que puede ser privado o no y que además recibe como parámetro el ID del post que deseamos consultar. Internamente manejaremos las excepciones y lógica adicional de consulta que consideremos pertinente. La forma más básica de este método es la siguiente.

```typescript
async findOne(id: number) {
  const post = await this.postRepository.findOne({
    where: { id },
    relations: {
      user: true
    }
  });

  if (!post) {
    throw new NotFoundException(`Post with id ${id} not found`);
  }
  return post;
}
```

Es importante observar que en este caso podemos decirle a TypeORM que al momento de hacer la consulta, el resultado incluya los campos de las relaciones, por ejemplo `user`.

De igual forma, si deseamos obtener todos los registros de la tabla, podemos definir el siguiente método `findAll()`:

```typescript
async findAll() {
  const posts = await this.postRepository.find({
    relations: {
      user: true
    }
  });
  return posts;
}
```

### Acción de eliminación {#delete}

Podemos decir que la acción de borrado es la más sencilla de todas, en el sentido de que solo debemos obtener el ID del post que deseamos eliminar y pasarlo al método `remove` del Repository:

```typescript
async remove(id: number) {
  await this.findOne(id);
  await this.postRepository.delete(id);
  return { message: `Post with id ${id} has been deleted` };
}
```

Observa que aunque no es estrictamente necesario consultar el post, utilizamos el método `findOne()` par asegurarnos de que el post existe antes de intentar borrarlo. Es solo una medida de precaución y buenas prácticas.

Espero que el contenido de esta entrada te haya resultado útil, nos vemos en el próximo post donde abordaremos el siguiente tipo de relación. La relación **Uno a Muchos**.

---

Autor: Julio César Echeverri M.
