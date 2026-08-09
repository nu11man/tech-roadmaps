---
layout: default
title: Relaciones Uno a Uno en TypeORM y NestJS
---

# Relaciones Uno a Uno en TypeORM y NestJS

En el artículo anterior de creación de entidades con TypeORM vimos como ingresar datos en una tabla. En este artículo veremos como construir y gestionar relaciones _uno a uno_ entre dos entidades. Para esto daremos continuidad al ejemplo de una entidad **User** pero agregaremos una nueva entidad **Profile**. Cada entidad User debe tener una y _solo una_ entidad Profile asociada y viceversa.
