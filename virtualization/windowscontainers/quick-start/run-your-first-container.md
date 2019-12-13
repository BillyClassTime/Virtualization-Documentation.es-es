---
title: Contenedores de Windows y Linux en Windows 10
description: Inicio rápido de implementación de contenedores
keywords: Docker, contenedores, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909575"
---
# <a name="get-started-run-your-first-windows-container"></a>Introducción: ejecución del primer contenedor de Windows

En este tema se describe cómo ejecutar el primer contenedor de Windows, después de configurar el entorno, tal como se describe en [Introducción a la preparación de Windows para contenedores](./set-up-environment.md). Para ejecutar un contenedor, primero debe instalar una imagen base, que proporciona una capa básica de servicios del sistema operativo para el contenedor. A continuación, cree y ejecute una imagen de contenedor, que se basa en la imagen base. Para obtener más información, lea.

## <a name="install-a-container-base-image"></a>Instalación de una imagen base de contenedor

Todos los contenedores se crean a partir de imágenes de contenedor. Microsoft ofrece varias imágenes de inicio, denominadas imágenes base, entre las que puede elegir (para más información, consulte [imágenes base de contenedor](../manage-containers/container-base-images.md)). Este procedimiento extrae (descarga e instala) la imagen base de nano Server ligera.

1. Abra una ventana del símbolo del sistema (como el símbolo del sistema integrado, PowerShell o [Windows terminal](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)) y, a continuación, ejecute el siguiente comando para descargar e instalar la imagen base:

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > Si ve un mensaje de error que indica `no matching manifest for unknown in the manifest list entries`, asegúrese de que Docker no está configurado para ejecutar contenedores de Linux.

2. Una vez finalizada la descarga de la imagen, lea el [CLUF](../images-eula.md) mientras espera: Compruebe que existe en el sistema mediante una consulta al repositorio de imágenes de Docker local. Al ejecutar el comando `docker images` se devuelve una lista de imágenes instaladas.

   Este es un ejemplo de la salida que muestra la imagen de nano Server.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Ejecución de un contenedor de Windows

En este sencillo ejemplo, se creará e implementará una imagen de contenedor "Hola mundo". Para obtener la mejor experiencia, ejecute estos comandos en una ventana de símbolo del sistema con privilegios elevados (pero no use el Windows PowerShell ISE; no funciona para las sesiones interactivas con contenedores, a medida que los contenedores parezcan bloqueados).

1. Inicie un contenedor con una sesión interactiva desde la imagen de `nanoserver` escribiendo el siguiente comando en la ventana del símbolo del sistema:

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. Una vez iniciado el contenedor, la ventana del símbolo del sistema cambia el contexto al contenedor. Dentro del contenedor, vamos a crear un archivo de texto "Hola mundo" simple y, a continuación, salir del contenedor escribiendo los siguientes comandos:

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. Obtenga el identificador de contenedor para el contenedor que acaba de salir ejecutando el comando [Docker PS](https://docs.docker.com/engine/reference/commandline/ps/) :

   ```console
   docker ps -a
   ```

4. Cree una nueva imagen "HelloWorld" que incluya los cambios en el primer contenedor que ejecutó. Para ello, ejecute el comando [Docker commit](https://docs.docker.com/engine/reference/commandline/commit/) , reemplazando `<containerid>` por el identificador del contenedor:

   ```console
   docker commit <containerid> helloworld
   ```

   Una vez finalizado, tendrá una imagen personalizada que contiene el script de hola a todos. Esto puede verse con el comando [Docker images](https://docs.docker.com/engine/reference/commandline/images/) .

   ```console
   docker images
   ```

   A continuación presentamos un ejemplo de la salida:

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. Por último, ejecute el nuevo contenedor mediante el comando [Docker Run](https://docs.docker.com/engine/reference/commandline/run/) con el parámetro `--rm` que quita automáticamente el contenedor una vez que se detiene la línea de comandos (cmd. exe).

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   El resultado es que se ha creado un contenedor a partir de la imagen ' HelloWorld ', se ha iniciado una instancia de cmd. exe en el contenedor que leyó el archivo y que se ha generado el contenido del archivo en el Shell y, a continuación, el contenedor se ha detenido y se ha quitado.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtenga información sobre cómo incluir en un contenedor una aplicación de ejemplo](./building-sample-app.md)
