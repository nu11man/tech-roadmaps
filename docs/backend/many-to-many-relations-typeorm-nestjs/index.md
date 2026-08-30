---
layout: default
title: Relaciones Muchos a Muchos en TypeORM y NestJS
---

# Relaciones Muchos a Muchos en TypeORM y NestJS

Si hemos llegado a este punto es porque ya vimos como configurar TypeORM en NestJS, creamos entidades, aprendimos a gestionar relaciones uno a uno, uno a muchos y finalmente, en esta entrada vamos a abordar el tipo final de relaciones, la relación **muchos a muchos**.

#### Contenido

- [Definición de las entidades](#definicion-de-entidades)
- [Actualización de DTO](#actualizar-dtos)
- [Actualización de servicios](#actualizar-servicios)
- [Registrar las Entidades en el Módulo](#inyeccion-entidades)
- [Acción de Creación](#create)
- [Acción de Actualización](#update)
- [Acción de Consulta](#read)
- [Acción de Eliminación](#delete)

---

### Definición de las entidades {#definicion-de-entidades}

De las entradas anteriores ya habíamos creado las entidades _User_, _Profile_ y _Post_. Esas entidades presentaban las siguientes relaciones:

- _User_ y _Profile_: Son entidades con relación _one to one_ porque un perfil solo puede estar asociado a un usuario y a su vez un usuario solo puede tener asociado un perfil.

- _User_ y _Post_: Tienen una relación _one to many_ porque un usuario puede tener varios post asociados, pero un post solo puede tener asignado un usuario.

Ahora, vamos a crear la entidad _Category_ y vamos a relacionarla con la entidad _Post_. En este caso la relación que obtendremos será _many to many_ ya que un post puede tener asociadas varias categorías y una categoría puede estar asignada a varios posts.

Definamos la entidad _Category_ con un atributo _name_ y los campos usuales de _primary key_, _create date_ y _update date_. Sin embargo, ahora vamos a ver el uso de un nuevo decorador:

```typescript
import { Entity, ManyToMany } from 'typeorm';
import { Post } from './post.entity';

@Entity({ name: 'categories' })
export class Category {

  ...

  @ManyToMany(() => Post, (post) => post.categories)
  posts!: Post[];
}
```

En esta definición de la entidad `Category` usamos el nuevo decorador `@ManyToMany()` que funciona en dos sentidos dependiendo de la entidad en la que se use. En este caso tenemos una referencia simple, es decir, indicamos que el campo `posts` apunta a una colección de Posts.

Ahora debemos actualizar la entidad `Post` que utilizará además del decorador `@ManyToMany()` un decorador adicional que parametriza la tabla que relaciona los múltiples posts con las múltiples categorias, a veces llamada "Tabla ternaria" o "Tabla intermedia".

```typescript
import { ManyToMany, JoinTable } from 'typeorm';
import { Category } from './category.entity';

@Entity({ name: 'posts' })
export class Post {
  ...

  @ManyToMany(() => Category, (category) => category.posts)
  @JoinTable({
    name: 'post_categories',
    joinColumn: { name: 'post_id', referencedColumnName: 'id' },
    inverseJoinColumn: { name: 'category_id', referencedColumnName: 'id' }
  })
  categories!: Category[];
}
```

Observa que en esta entidad usamos lo siguiente:

- `@ManyToMany()`: Indica que en este campo referenciamos una lista de entidades tipo `@Category`.
- `@JoinTable()`: Con este decorador parametrizamos la tabla intermedia que relaciona los múltiples Posts con las múltiples Categories. Los tres campos principales son: El nombre de la tabla intermedia, la columna que funciona como llave en esta entidad (`joinColumn`) y la columna que funciona como llave en la otra entidad de esta relación (`inverseJoinColumn`).

Observa que en ambas entidades tenemos dos arrays.

### Actualización de DTO {#actualizar-dtos}

Es importante tener presente que en los DTOs no vamos a indicar una lista de objetos sino una lista de identificadores (`id`) tanto en category como en post.

Para el DTO de creación de Posts tenemos entonces:

```typescript
import { IsArray, IsNumber, IsOptional } from 'class-validator';

export class CreatePostDto {

  ...

  @IsArray()
  @IsNumber({}, { each: true })
  @IsOptional()
  categoryIds?: number[];
}
```

Para el DTO de creación de Categories tenemos:

```typescript
import { IsArray, IsNumber, IsOptional } from 'class-validator';

export class CreateCategoryDto {

  ...

  @IsArray()
  @IsNumber({}, { each: true })
  @IsOptional()
  postsIds?: number[];
}
```

### Actualización de servicios {#actualizar-servicios}

### Registrar las entidades en el módulo {#inyeccion-entidades}

### Acción de creación {#create}

Para crear una nueva entidad en la base de datos debemos tener presente el mapeo de los IDs que relacionan la entidad actual con la entidad de la otra tabla. Por ejemplo, para crear un nuevo Posts:

```typescript
async create(post: CreatePostDto) {

  const newPost = await this.postRepository.save({
    ...post,
    user: { id: post.userId },
    categories:
      post.categoryIds?.map((categoryId) => ({ id: categoryId })) || []
  });

  const createdPost = await this.findOne(newPost.id);
  return createdPost;
}
```

### Acción de actualización {#update}

Ahora si deseamos realizar la actualización de los IDs en el array que mantiene la relación, podemos abordarlo de la siguiente manera:

```typescript

```

### Acción de consulta {#read}

Observemos que aunque la inyección del respository pattern se mantiene como en las relaciones estudiadas anteriormente, aquí debemos indicar a TypeORM si deseamos que nos resuelva la lista de componentes que referenciamos (en este caso categorias), en caso de ser así, tendríamos una función como se muestra a continuación:

```typescript
async findOne(id: number) {
    const post = await this.postRepository.findOne({
      where: { id },
      relations: {
        user: true,
        categories: true
      }
    });
    if (!post) {
      throw new NotFoundException(`Post with id ${id} not found`);
    }
    return post;
  }
```

### Acción de eliminación {#delete}

Espero que el contenido de esta entrada te haya resultado útil, nos vemos en el próximo post donde abordaremos el siguiente tipo de relación. La relación **Uno a Muchos**.

---

Autor: Julio César Echeverri M.
