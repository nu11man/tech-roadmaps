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

## Setup de Ruby


## Setup de Java


## Setup de Python