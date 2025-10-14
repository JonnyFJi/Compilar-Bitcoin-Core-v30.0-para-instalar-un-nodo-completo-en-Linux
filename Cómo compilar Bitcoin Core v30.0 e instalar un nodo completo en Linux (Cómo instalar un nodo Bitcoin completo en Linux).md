---
title: Cómo compilar Bitcoin Core v30.0 e instalar un nodo completo en Linux
tags: [bitcoin, cli, nodo, node, macos]

---

# Cómo instalar un nodo Bitcoin completo en Linux
###### tags: `bitcoin` `cli` `nodo` `Core` `Linux`

Tutorial para compilar **Bitcoin Core** desde el código fuente para entornos que requieren mayor control, transparencia o simplemente cuando no existen binarios oficiales para la plataforma deseada.
Esto permite verificar e incluso modificar el código antes de utilizarlo en producción.
- El proceso estándar se realiza habitualmente en sistemas *Linux*, pero se puede adaptar para *macOS* y, con ciertas herramientas, también para *Windows*.

**Tabla de contenido**
[TOC]

## Autores
Jonny_Ji. 
Twitter para correcciones, comentarios o sugerencias: [@JonnyJi50127056](https://x.com/JonnyJi50127056)

El presente tutorial fue elaborado para el curso [Nodes from zero to hero](https://libreriadesatoshi.com/courses/nodes-from-zero-to-hero) a través de [@libreriadesatoshi](https://twitter.com/libdesatoshi).

## Cómo empezar
En el siguiente enlace puedes encontrar la documentación de referencia:
- [How to install a full Bitcoin node on a testnet network with Linux Ubuntu](https://hackmd.io/SFuZ3YXfQWSX9tUkWEghHw?both=#How-to-install-a-full-Bitcoin-node-on-a-testnet-network-with-Linux-Ubuntu)
- [Compiling Bitcoin Core from the Source Code](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch03_bitcoin-core.adoc#compiling-bitcoin-core-from-the-source-code)

## Tipos de instalación
Hay cuatro formas de configurar un nodo de Bitcoin, las enumeraré comenzando desde la más fácil hasta la más difícil:

1. Con software plug & play como Umbrel, Star9 o similares.
Estos programas tienen instaladores para Raspberry Pi y también se pueden instalar como sistema operativo en PC nuevas o usadas.
En el caso de Umbrel y Star9, se instalan como Sistemas Operativos que ya vienen con programas pre-configurados como Bitcoin Core y varias implementaciones de la red Lightning de forma predeterminada. Lo que facilita que los usuarios no técnicos ejecuten un nodo de una manera muy sencilla y con privacidad de forma predeterminada.
1. Instalar desde contenedores Docker. Lo cual requiere algunos conocimientos más técnicos.
Para construir un contenedor Docker de Bitcoin Core, puedes utilizar una imagen preconfigurada, que sea ampliamente utilizada y revisada para garantizar su integridad.
Alternativamente, puedes crear tu propia imagen personalizada utilizando un Dockerfile que incluya el cliente Bitcoin Core oficial, especificando la versión deseada y configurando los archivos de configuración necesarios
1. Instalación a partir de binarios ya compilados. En *bitcoincore.org* existen instaladores binarios para varios sistemas operativos, es cuestión de descargarlos y seguir las instrucciones para instalarlo en tu equipo.
1. Descargue el código fuente, compílarlo e instálarlo.

En este tutorial vamos a desarrollar el `punto 4`, que puede resultar un desafío para personas no técnicas. Si eres un usuario técnico o no tan técnico pero estás ansioso por aprender, este tutorial es para ti.

Como requisito para este tutorial, debes haber instalado previamente una versión de `Linux`, en nuestro caso la vamos a basar en `Ubuntu`, ya que es una distribución ampliamente utilizada por la comunidad de desarrolladores de Bitcoin y hay una base bastante amplia de información y tutoriales basados en esta distribución.

## Instalar programas, bibliotecas y dependencias

**Programas que necesitamos**

Desde la terminal, vamos a instalar estos dos programas que necesitaremos más adelante:
```shell
sudo apt-get install git curl
```

**Bibliotecas y dependencias**

Ahora necesitamos instalar algunas bibliotecas. Al instalar bibliotecas, a veces puedes enumerar muchas en un solo comando y separarlas con un solo espacio. En este tutorial, los dividí en grupos similares a la documentación de compilación Github para [Github for Debian/Ubuntu](https://github.com/bitcoin/bitcoin/blob/master/doc/build-unix.md)

Ejecuta los siguientes comandos para instalar todas las librerías que te ayudarán a compilar el código bitcoind y que también te serán útiles en caso de que quieras utilizar herramientas para aprender a desarrollar.


```shell
sudo apt update
sudo apt-get install build-essential autoconf libtool autotools-dev automake pkg-config bsdmainutils python3 
sudo apt-get install libevent-dev libboost-dev
sudo apt-get install libsqlite3-dev
sudo apt-get install libminiupnpc-dev libnatpmp-dev
sudo apt-get install libzmq3-dev
sudo apt-get install libssl-dev libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-test-dev libboost-thread-dev libprotobuf-dev protobuf-compiler git cmake
```

Si quieres utilizar GUI (interfaz gráfica) para administrar bitcoind debes instalar estas librerías, si solo vas a utilizar la línea de comandos no es necesario.

```shell
sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools
```

## Compilar e instalar bitcoind

Verificamos que estamos en el home del usuario actual:

```shell
cd ~
```

Y ejecutamos este comando para descargar el código del repositorio de Bitcoin:

```shell
git clone https://github.com/bitcoin/bitcoin.git
```

Una vez finalizada la descarga, cambiamos a la carpeta `bitcoin` que acaba de crearse ejecutando el comando:

```shell
cd bitcoin
```
![Captura de pantalla de 2025-10-11 08-20-57](https://hackmd.io/_uploads/SyDL36vTgx.png)

Miramos las etiquetas para ver las versiones disponibles con este comando:

```shell
git tag
```

![Captura de pantalla de 2025-10-11 23-19-44](https://hackmd.io/_uploads/HJeqA5_agl.png)

También puedes ver la última versión publicada en esta URL:
[https://github.com/bitcoin/bitcoin/releases](https://github.com/bitcoin/bitcoin/releases)
![Captura de pantalla de 2025-10-11 07-05-25](https://hackmd.io/_uploads/rkAYT3Ppxg.png)
- Para el momento de escribir este tutorial, la versión 29.1 de Bitcoin Core es la última publicada.

Cambiamos a la rama o etiqueta de la versión estable que queremos instalar.

- En este caso vamos a pasar a la última versión definitiva, para ello ejecutamos este comando:
```shell
git checkout v30.0
```

![Captura de pantalla de 2025-10-11 23-23-00](https://hackmd.io/_uploads/SJAVyiuaee.png)


**Iniciamos el proceso de compilación**

Para compilar Bitcoin Core desde el código fuente, en las versiones más recientes el proceso ha evolucionado respecto a versiones anteriores, incluyendo recientemente el uso de `CMake` para construcción y pruebas. 

**Configurar y generar archivos con CMake**

Crear el directorio `build` separado dentro de la carpeta `bitcon`:

```shell
mkdir build
cd build
```

Ahora vamos a preparar el entorno para compilar el software de referencia Bitcoin Core, para ello ejecutamos el siguiente comando **cmake** con el prefijo para ignorar la base de datos BDB incompatible y el soporte de multiprocesos, cada prefijo que agregan al comando, activara o desactivara alguna función, incluso puede agregar funciones extras al programa (como vamos a ejecutar `bitcoind`, no necesitamos `bitcoin-qt`):
```shell
cmake .. -DWITH_INCOMPATIBLE_BDB=ON -DENABLE_IPC=OFF
```

Si todo sale bien, se deberían haber creado los scripts para compilar `bitcoind`.
Si no, revise qué bibliotecas faltaban, instálelas e inténtelo nuevamente.

- Al finalizar, verá una imagen como esta:

![Captura de pantalla de 2025-10-11 09-05-00](https://hackmd.io/_uploads/Sk2S8CDTgx.png)
![Captura de pantalla de 2025-10-11 09-05-24](https://hackmd.io/_uploads/rkAUUAvalx.png)

Ahora sí, vamos a compilar y para ello ejecutamos el comando **make**. (Vamos a ejecutar make usando todos los núcleos disponibles para acelerar el proceso):
```shell
make -j$(nproc)
```
`make` se puede ejecutar sin parámetros y usa un solo núcleo del procesador, pero si usamos el parámetro `-j$(nproc)`, usaremos todos los núcleos del procesador para que la compilación se realice en paralelo y tome menos tiempo.

Este proceso puede tardar varios minutos o entre una hora y una hora y media, dependiendo de los núcleos de tu CPU y la memoria disponible. Te recomiendo que no tengas muchas aplicaciones abiertas.

- Al final, podemos ver una imagen similar a esta:

![Captura de pantalla de 2025-10-11 09-45-52](https://hackmd.io/_uploads/Hkgl2k_pge.png)
![Captura de pantalla de 2025-10-11 10-36-52](https://hackmd.io/_uploads/rJYS3JOTle.png)

Si no hubo problemas la compilación se habrá creado, así que ahora vamos a proceder con la instalación, ejecutando el comando:

```shell
sudo make install
```

- La instalación dejará una salida de consola como la que se ve en la siguiente imagen:

![Captura de pantalla de 2025-10-11 10-44-42](https://hackmd.io/_uploads/Sk77AyOagl.png)
![Captura de pantalla de 2025-10-11 10-44-51](https://hackmd.io/_uploads/ByG4RyOTgx.png)

Esto debería instalar el binario `bitcoind` en la ruta: `/usr/local/bin/bitcoind` así como `/usr/local/bin/bitcoin-cli` y `/usr/local/bin/bitcoin-qt` en caso de que hayas decidido instalar Bitcoin Core con la interfaz gráfica (GUI).

**Verificar Versión**
Podemos verificar la versión de `bitcoind` que acabamos de instalar, ejecutando el demonio Bitcoin Core:
```shell
cd
bitcoind --version
```
## Ejecutar bitcoind
**Configurar el archivo bitcoin.conf**
Cuando se ejecuta `bitcoind` por primera vez, un directorio `.bitcoin` se crea en la carpeta personal del usuario que contiene los archivos de trabajo del programa y el archivo `bitcoin.conf`.
Ya que aún no hemos iniciado el nodo con `bitcoind`, vamos a crear ese archivo de antemano, para agregar la configuración con la que queremos que funcione Bitcoin Core.
Para esto ejecutamos estos comandos que nos ayudarán creando `bitcoin.conf`.
- Creamos el archivo de configuración con cualquier editor de texto. Usaré `nano` para simplificar:

```shell
cd .bitcoin
sudo nano bitcoin.conf
```
Vamos a utilizar una configuración para instalar un nodo que contenga el historial completo de los bloques, ya que si bien podría ser un nodo podado (un nodo que no contiene el historial completo de los bloques), es muy recomendable hacerlo con un nodo que tenga todo el historial y así aprovechar el rendimiento y consumo de recursos del nodo.

Se pueden agregar muchos parámetros al archivo de configuración, pero para simplificar las cosas solo vamos a introducir un par de líneas que indicarán que vamos a utilizar la red de `signet`.
  
```shell=
signet=1
txindex=1
```
- Antes de correr `bitcoind` en segundo plano, ten en cuenta que comenzará a descargarse la blockchain `signet`, que al momento de crear este tutorial pesa aproximadamente 20Gigas, lo que tomará un tiempo para sincronizar.

**Ejecute bitcoind y descargue la cadena de bloques**
Ahora corre `bitcoind` como un demonio para que se ejecute en segundo plano y comience a descargar la cadena de bloques:
```shell
bitcoind -daemon
```
Para interactuar con `bitcoind`, debes utilizar `bitcoin-cli` (Bitcoin interfaz de línea de comandos) seguido del comando que desea ejecutar. Utilizando el comando `help`puedes encontrar una lista de ordenes para ejecutar con el nodo.
```shell
bitcoin-cli help
```
Para comprobar el progreso del nodo, utilice este comando:
```shell
bitcoin-cli getblockchaininfo
```
![Captura de pantalla de 2025-10-11 12-19-16](https://hackmd.io/_uploads/H11zNW_Tll.png)

- En el `verificationprogress`, el parámetro indicará el porcentaje de progreso de la sincronización de la cadena de bloques, cuando llegue a 99% ya habrá descargado la blockchain completa.
- Con 6 horas al día en las que su nodo completo puede dejarse funcionando es suficiente para mantenerlo sincronizado. (Puedes hacer otras cosas con tu computadora mientras ejecutas un nodo completo). Más horas sería mejor, y lo mejor de todo sería si pudieras correr tu nodo continuamente, 24/7 es lo ideal para apoyar la red.

**Nota**: Muchos sistemas operativos actuales (Windows, Mac y Linux) entran en una modo de bajo consumo después de que se activa el protector de pantalla, esto desacelera o detiene el tráfico de red. Esta suele ser la configuración predeterminada en computadoras portátiles y en todas las computadoras portátiles y de escritorio MacOS X. Comprueba la configuración de tu protector de pantalla y desactive las opciones automáticas de “dormir” o “suspender” para asegurarse. Apoye la red siempre que su computadora esté en funcionamiento.

Si has llegado hasta aquí, felicitaciones, ahora tienes un nodo Bitcoin listo para usar. Este es el primer pilar para comenzar a construir tu soberanía monetaria.

# Team "Librería de Satoshi"