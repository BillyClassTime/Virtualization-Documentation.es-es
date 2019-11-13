---
title: Contenedores de Windows y Linux en Windows 10
description: Inicio rápido de implementación de contenedores
keywords: acoplador, contenedores, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288133"
---
# <a name="get-started-run-your-first-windows-container"></a>Introducción: ejecutar el primer contenedor de Windows

En este tema se describe cómo ejecutar el primer contenedor de Windows, después de configurar su entorno, como se describe en [Introducción: preparar Windows para contenedores](./set-up-environment.md). Para ejecutar un contenedor, primero debe instalar una imagen base, que proporciona una capa básica de servicios del sistema operativo a su contenedor. A continuación, crea y ejecuta una imagen de contenedor, que se basa en la imagen base. Para obtener más información, siga leyendo.

## <a name="install-a-container-base-image"></a>Instalar una imagen base del contenedor

Todos los contenedores se crean a partir de imágenes de contenedor. Microsoft ofrece varias imágenes de inicio, denominadas imágenes base, entre las que elegir (para obtener más información, consulte [imágenes base del contenedor](../manage-containers/container-base-images.md)). Este procedimiento extrae la imagen básica de nano Server (descarga e instala).

1. Abra una ventana del símbolo del sistema (como el símbolo del sistema integrado, PowerShell o [Windows terminal](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)) y, a continuación, ejecute el siguiente comando para descargar e instalar la imagen base:

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > Si ve un mensaje de error que dice `no matching manifest for unknown in the manifest list entries`, asegúrese de que el acoplador no esté configurado para ejecutar los contenedores de Linux.

2. Una vez terminada la descarga de la imagen, lea el [CLUF](../images-eula.md) mientras espera: para comprobar si existe la existencia en el sistema, consulte el repositorio local de la imagen del Dock. La ejecución del `docker images` comando devuelve una lista de las imágenes instaladas.

   Este es un ejemplo de la salida que muestra la imagen de nano Server.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Ejecutar un contenedor de Windows

Para este sencillo ejemplo, se creará e implementará una imagen de contenedor de "Hola a todos". Para obtener la mejor experiencia, ejecute estos comandos en una ventana de símbolo del sistema con privilegios elevados (pero no use Windows PowerShell ISE; no funciona para sesiones interactivas con contenedores, ya que los contenedores parecen bloquearse).

1. Inicie un contenedor con una sesión interactiva desde `nanoserver` la imagen escribiendo el comando siguiente en la ventana del símbolo del sistema:

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. Después de iniciar el contenedor, la ventana del símbolo del sistema cambia el contexto al contenedor. Dentro del contenedor, crearemos un simple archivo de texto "Hola a todos" y, a continuación, saliremos del contenedor escribiendo los siguientes comandos:

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. Para obtener el identificador del contenedor que acaba de salir, ejecute el comando [PS del Dock](https://docs.docker.com/engine/reference/commandline/ps/) :

   ```console
   docker ps -a
   ```

4. Cree una nueva imagen ' HelloWorld ' que incluya los cambios en el primer contenedor que ejecutó. Para ello, ejecute el comando de [confirmación del acoplador](https://docs.docker.com/engine/reference/commandline/commit/) , `<containerid>` que se reemplaza por el identificador del contenedor:

   ```console
   docker commit <containerid> helloworld
   ```

   Una vez finalizado, tendrá una imagen personalizada que contiene el script de hola a todos. Esto se puede ver con el comando [imágenes del acoplador](https://docs.docker.com/engine/reference/commandline/images/) .

   ```console
   docker images
   ```

   A continuación presentamos un ejemplo de la salida:

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. Por último, ejecute el nuevo contenedor con el comando [Ejecutar del acoplador](https://docs.docker.com/engine/reference/commandline/run/) con `--rm` el parámetro que quita automáticamente el contenedor una vez que se detiene la línea de comandos (cmd. exe).

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   El resultado es que se creó un contenedor a partir de la imagen ' HelloWorld ', una instancia de cmd. exe se inició en el contenedor que leyó el archivo y ha generado el contenido del archivo en el Shell y, a continuación, el contenedor se ha detenido y se ha quitado.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre cómo entienda una aplicación de ejemplo](./building-sample-app.md)
