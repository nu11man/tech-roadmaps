---
layout: default
title: Relaciones Muchos a Muchos en TypeORM y NestJS
---

# Relaciones Muchos a Muchos en TypeORM y NestJS

Si hemos llegado a este punto es porque ya vimos como configurar TypeORM en NestJS, creamos entidades, aprendimos a gestionar relaciones uno a uno, uno a muchos y finalmente, en esta entrada vamos a abordar el tipo final de relaciones, la relación **muchos a muchos**.

#### Contenido

- [Definición de las entidades](#definicion-de-entidades)
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

Definamos la entidad _Category_ con un atributo _name_ y los campos usuales de _primary key_, _create date_ y _update date_. Sin embargo, ahora vamos a ver el uso de nuevos decoradores:

```typescript

```

### Registrar las entidades en el módulo {#inyeccion-entidades}

### Acción de creación {#create}

### Acción de actualización {#update}

### Acción de consulta {#read}

### Acción de eliminación {#delete}

Espero que el contenido de esta entrada te haya resultado útil, nos vemos en el próximo post donde abordaremos el siguiente tipo de relación. La relación **Uno a Muchos**.

---

Autor: Julio César Echeverri M.
