---
layout: default
title: Configuración Java, NodeJS y Ruby en Ubuntu
---

# Configuración Java, NodeJS y Ruby en Ubuntu

Uno de los aspectos diferenciadores entre un desarrollador con experiencia y uno que apenas inicia en el mundo del software es la gestión de su ambiente de desarrollo. No hay nada más frustrante y que te robe más tiempo que trabajar sobre un ambiente frágil e inestable.

¿Te imaginas decirle a tus clientes o compañeros "que raro, a mi no me funciona"?, porque tienes instalada una versión de Java, Node, Ruby o Python que no son compatibles y que además no puedes modificar.

En este artículo vamos a configurar nuestro sistema con máxima flexibilidad para gestionar las versiones de cada uno de los intérpretes que mencioné antes.

---

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
Ruby es un lenguaje muy utilizado y cuenta con una gran colección de *gemas* (el equivalente a paquetes en Node). Contamos con un par de herramientas para gestionar las versiones del intérprete de Ruby y veremos como configurarlas y usarlas a continuación, iniciando con el instalador de Ruby.

En primer lugar instalamos las dependencias:

```bash
sudo apt update
sudo apt install -y build-essential wget curl
```

A continuación, vamos al directorio de archivos temporales de nuestro sistema, descargamos el proyecto de github y ejecutamos el comando de instalación. Esto lo logramos con las siguientes instrucciones:

**Nota**: Recuerda revisar la versión más actual [en el repositorio oficial del proyecto](https://github.com/postmodern/ruby-install#install) y reemplazarlo en los siguientes comandos.

```bash
wget https://github.com/postmodern/ruby-install/releases/download/v0.10.2/ruby-install-0.10.2.tar.gz
tar -xzvf ruby-install-0.10.2.tar.gz
cd ruby-install-0.10.2/
sudo make install
```
La instalación es inmediata, ahora solo debes reiniciar la terminal de comandos y para verificar la instalación exitosa, puedes verificar la versión instalada:

```bash
ruby-install --version
```

Luego de agregar la herramienta con la que instalamos las versiones de Ruby, vamos a instalar ahora el administrador de versiones **chruby**, que como podemos deducir, es el encargado de gestionar cuál versión estamos usando en cada momento. Para ello ejecutaremos las siguientes instrucciones:

**Nota**: Recuerda ir al [repositorio del proyecto](https://github.com/postmodern/chruby#install) y revisar el número de la última versión de `chruby` para reemplazarla en los siguientes comandos.

```bash
cd /tmp
wget https://github.com/postmodern/chruby/releases/download/v0.3.9/chruby-0.3.9.tar.gz
tar -xzvf chruby-0.3.9.tar.gz
cd chruby-0.3.9/
sudo make install
```

Una vez ejecutados los pasos anteriores, debemos ir al archivo `.zshrc` o `.bashrc` y agregar la siguiente línea:

```bash
source /usr/local/share/chruby/chruby.sh
```

Ahora reiniciamos la terminal y verificamos que la instalación de `chruby` fue correcta ejecutando el siguiente comando:

```bash
chruby --version
```

Finalmente podemos indicar una versión que se use por defecto en cada sesión y también activar la caracterísca de detección de versión en nuestros proyectos, esto es, que cuando ingresamos a una carpeta y se detecta el archivo `.ruby-version` automáticamente comenzamos a usar esa versión. Todo esto lo logramos agregando lo siguiente al archivo `.zshrc` o `.bashrc`.

```bash
chruby ruby-1.9 # a desired default version
source /usr/local/share/chruby/auto.sh
```

Al final, reiniciamos la terminal.

### Como usar `ruby-install` y `chruby`

En primer lugar, para usar alguna versión de Ruby debemos tenerla instalada, `ruby-install` ofrece las siguientes opciones (más utilizadas):

- `ruby-install`: Lista las versiones de Ruby soportadas y las versiones estables.
- `ruby-install ruby`: Instala la versión estable más actual de Ruby.
- `ruby-install ruby <version>`: Instala una versión específica de Ruby.

**Nota**: Después de instalar una nueva versión de Ruby, es importante reiniciar la terminal para que las demás herramientas la detecten.

Luego para moverse entre versiones instaladas de Ruby, nos valemos de las opciones de `chruby`:

- `chruby`: Lista las versiones instaladas de Ruby.
- `chruby ruby-<version>`: Activa la versión específica de Ruby que le indiquemos.


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