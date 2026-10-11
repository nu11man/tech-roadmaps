---
layout: default
title: Seguridad de rutas con Api-Key en NestJS
---

# Seguridad de rutas con Api-Key en NestJS

En un artículo anterior habíamos analizado la protección de rutas utilizando Tokens JWT. Sin embargo, esta no es la única estrategia de protección. En este artículo vamos a ver como agregar validación de llaves usando la estrategía API-KEY y, además, vamos a ver como ajustar nuestra configuración de Swagger para que este tipo de protección se vea reflejado en la documentación.

#### Contenido

- [Generar y configurar la api-key](#generar-configurar-api-key)
- [Crear un api-key Guard](#crear-api-key-guard)
- [Proteger las rutas con api-key](#proteger-rutas-con-api-key)
- [Integrar Swagger y api-keys](#integrar-swagger-api-key)

---

### Generar y configurar la api-key {#generar-configurar-api-key}

En primer lugar debemos generar una api key, esto lo logramos ejecutando la siguiente línea en nuestra terminal de comandos:

```bash
node -e "console.log(require('crypto').randomUUID())"
```

Esto nos retornará una cadena de texto como la siguiente:

```bash
d6d4f9ce-1dd2-4859-8a9f-b23bfd4bd67e
```

Ahora vamos a almacenar este cadena en nuestro archivo `.env` con el nombre `API_KEY`.
Luego, debemos actualizar el `config loader`, el `schema validation` y la interfaz que describe el modelo.

### Crear un api-key Guard {#crear-api-key-guard}

Con el secreto creado y la configuración de ambiente actualizada, vamos a crear el archivo `api-key.guard.ts` en la ruta que corresponda, por ejemplo, si vamos a proteger un módulo específico podemos crear un directorio de la forma `module_name/guards/api-key.guard.ts`. Por otra parte

### Proteger las rutas con api-key {#proteger-rutas-con-api-key}

### Integrar Swagger y api-keys {#integrar-swagger-api-key}

---

Autor: Julio César Echeverri Marulanda
