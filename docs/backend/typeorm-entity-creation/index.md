---
layout: default
title: Creación de Entidades y Repository Pattern con TypeORM
---

# Creación de Entidades y Repository Pattern con TypeORM

En el artículo anterior vimos como instalar y configurar TypeORM en nuestro proyecto de NestJS. En esta oportunidad vamos a ver como crear una entidad de base de datos y acercarnos cada vez más al objetivo final, una aplicación de backend completamente funcional y lista para un ambiente productivo.

#### Contenido

- [Creación de la entidad](#creacion-de-entidades)
- [Respository Pattern](#repository-pattern)
- [Operación CREATE](#how-to-create)
- [Operación UPDATE](#how-to-read)
- [Operación DELETE](#how-to-delete)
- [Cómo consultar datos](#how-to-read-data)

---

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

- `Entity()`: Es el decorador que define la entidad que corresponde a una tabla de la base de datos. Normalmente los nombres de las tablas se escriben en plural y con \_ cuando corresponde a nombres compuestos. Este decorador recibe un objeto como parámetro que describe (entre otras opciones) el nombre de la tabla.

- `Column()`: Este decorador describe

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
    name: 'updated_at',
    type: 'timestamptz',
    default: () => 'CURRENT_TIMESTAMP'
  })
  updatedAt!: Date;
}
```
