---
layout: default
title: Autenticación basada en token JWT
---

# Autenticación basada en token JWT

En el artículo anterior implementamos la autenticación de los usuarios usando la una _Local Strategy_ de PassportJS, esa local strategy particularmente se basa en la verificación de _email_ y _password_. Eso está muy bien, implementamos el endpoint de `/login` y si el email y la contraseña son correctos, retornamos la información del usuario. Sin embargo, nos hacen falta dos aspectos muy importantes.

- Un mecanismo que permita al usuario acceder a recursos protegidos sin tener que enviar constantemente el email y password.
- Implementar protección en diferentes recursos que no deben ser accedidos por entidades no autenticadas.

Para resolver estos dos aspectos, vamos a ver como podemos utilizar la especificación JWT (JSON Web Token) para entregarle al usuario el equivalente a una llave digital, y además implementaremos la protección de algunos endpoints.

#### Contenido

- [Instalación de dependencias](#instalacion-de-dependencias)
- [Generación de tokens JWT](#generacion-de-tokens-jwt)
- [Creación de JWT Strategy](#jwt-strategy)
- [Protección de endpoints](#proteccion-de-endpoints)

---

### Instalación de dependencias {#instalacion-de-dependencias}

### Generación de tokens JWT {#generacion-de-tokens-jwt}

### Creación de JWT Strategy {#jwt-strategy}

### Protección de endpoints {#proteccion-de-endpoints}
