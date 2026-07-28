---
layout: default
title: Variables de Entorno en NestJS
---

# Configuración del Ambiente de Desarrollo en Ubuntu

## Setup de NodeJS
NVM (node version manager) es un paquete de software que nos permite administrar las versiones de NodeJS en nuestro sistema operativo de forma dinámica, evitando colisiones entre proyectos que requieren versiones de Node diferentes (incluso incompatibles entre sí) sin tener que preocuparnos:

Para realizar la instalación de este paquete tenemos como requisito `wget` o `curl` ya que el instalador es un archivo de bash.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.6/install.sh | bash
```
o usando `wget`:

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.6/install.sh | bash
```

**Nota**: Este instalador funciona incluso si tienes configurada como shell ZSH, no tienes que cambiar nada en el comando de instalación anterior.

Ahora puedes reiniciar la terminal para que la herramienta quede activada.

### Cómo usar nvm

Esta herramienta cuenta con bastantes opciones, pero en el día a día podemos valernos de unas pocas opciones muy frecuentes, algunas son:

- `nvm ls-remote`: Con esta opción podemos listar las opciones disponibles para instalación.
- `nvm install <version>`: Nos permite instalar la versión deseada de Node, algunas opciones son: 
    - `--lts`: instala la última versión LTS disponible.
    - `x.y.z`: Instala una versión específica.
    - `node`: Instala la última versión disponible:
- `nvm list`: Lista las versiones instaladas y listas para usar.
- `nvm use <version>`: Permite indicar una versión específica para usar, aplican las mismas opciones del comando `install`. 

**Nota**: Dado que `npm` se instala automáticamente con Node, podemos adicionalmente instalar el gestor de paquetes `yarn` con la siguiente instrucción:

```bash
npm install --global yarn
```

Puedes verificar la instalación con:

```bash
yarn --version
```


## Setup de Ruby


## Setup de Java
Al igual que ocurre con NVM, los proyectos que contruimos con Java suelen ser algunas veces bastante sensibles a los cambios de versión. Para hacer la gestión de versiones mucho más simple, podemos contar con el gestor *SDKman*. Podemos instalarlo con alguna de las siguientes instrucciones dependiendo de la shell que uses:

*Usando BASH*

```bash
curl -s "https://get.sdkman.io" | bash
```

*Usando ZSH*

```bash
curl -s "https://get.sdkman.io" | zsh
```

Después de ejecutar la instalación puedes reiniciar la terminal y ejecutar el comando `sdk help`.

### Cómo usar SDKman

SDKman es un gestor que puede instalar y gestionar múltiples SDK, como Java, Groovy, Scala, etc. Existen múltiples opciones pero las más usadas son las siguientes:

- `sdk help [command]`: Muestra como podemos usar este paquete.
- `sdk list java`: Muestra todas las opciones disponibles de Java para instalar.
- `sdk install java 17.0.20-zulu`: Instala una versión y distribución específica de Java, por ejemplo, esta es la versión  recomendada por React Native actualmente.
- `sdk use <version>`: Permite seleccionar una versión instalada previamente.

Para verficar que estás usando la versión deseada de Java puedes ejecutar las siguientes líneas en la terminal de comandos:

```bash
java --version
javac --version
```

## Setup de Python