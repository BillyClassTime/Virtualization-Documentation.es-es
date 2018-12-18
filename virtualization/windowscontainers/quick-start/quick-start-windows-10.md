---
title: Contenedores de Linux en Windows 10 y Windows
description: Inicio rápido de implementación de contenedores
keywords: docker, contenedores, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: dc500a7b6c0f8f078820407e6ed80ca5868bf4f3
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973655"
---
# <a name="windows-containers-on-windows-10"></a>Contenedores de Windows en Windows 10

> [!div class="op_single_selector"]
> - [Contenedores de Linux en Windows](quick-start-windows-10-linux.md)
> - [Contenedores de Windows en Windows](quick-start-windows-10.md)

Este ejercicio a través de crear y ejecutar contenedores de Windows en Windows 10.

En este inicio rápido llevará a cabo:

1. Instalar Docker para Windows
2. Ejecutar un contenedor simple de Windows

Este inicio rápido es específico de Windows10. En la tabla de contenido en el lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

## <a name="prerequisites"></a>Requisitos previos
Asegúrese de que cumples los requisitos siguientes:
- Un sistema de equipo físico que ejecuta Windows 10 Professional o Enterprise con actualización de aniversario (versión 1607) o una versión posterior. 
- Asegúrese de que [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) está habilitado.

***Aislamiento de Hyper-V:*** Contenedores de Windows Server requieren aislamiento de Hyper-V en Windows 10 para ofrecer a los desarrolladores con la misma versión de kernel y configuración que se usará en producción, más acerca de Hyper-V aislamiento puede encontrarse en la página de [contenedor acerca de Windows](../about/index.md) .

> [!NOTE]
> En la versión de Windows de actualización de octubre de 2018, ya no se impide a los usuarios se ejecuten un contenedor de Windows en el modo de aislamiento de procesos en Windows 10 Enterprise o Professional para fines de desarrollo o prueba. Consulta las [preguntas más frecuentes](../about/faq.md) para obtener más información.

## <a name="install-docker-for-windows"></a>Instalar a Docker para Windows

Descarga [Docker para Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) y ejecuta el programa de instalación (se le pedirá que inicie sesión. Crear una cuenta si no tienes uno ya). Se encuentran disponibles [instrucciones de instalación detalladas](https://docs.docker.com/docker-for-windows/install) en la documentación de Docker (en inglés).

## <a name="switch-to-windows-containers"></a>Cambiar a Windows contenedores

Después de la instalación, Docker para Windows ejecuta contenedores de Linux de forma predeterminada. Cambiar a contenedores de Windows mediante el menú de bandeja de Docker o ejecutando el siguiente comando en un PowerShell símbolo del sistema:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>Instalar imágenes base del contenedor

Los contenedores de Windows se crean a partir de imágenes base. El comando siguientes extraerá la imagen base de Nano Server.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

Una vez extraída, si se ejecuta `docker images`, se devolverá una lista de las imágenes instaladas, en este caso la imagen de Nano Server.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Lee el [CLUF](../images-eula.md)de imagen de sistema operativo de contenedores de Windows.

## <a name="run-your-first-windows-container"></a>Ejecutar el primer contenedor de Windows

Para este ejemplo simple, se crear e implementar una imagen de contenedor "Hello World". Para lograr una mejor experiencia ejecute estos comandos en un shell de CMD de Windows con privilegios elevados o en PowerShell.

> Windows PowerShell ISE no funciona en sesiones interactivas con contenedores. Aunque el contenedor se ejecuta, parece que se cuelga.

En primer lugar, inicie un contenedor con una sesión interactiva desde la imagen `nanoserver`. Una vez que se ha iniciado el contenedor, aparecerá un shell de comandos desde dentro del mismo.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

Dentro del contenedor se creará un archivo de texto sencillo de "Hello World".

```cmd
echo "Hello World!" > Hello.txt
```   

Una vez completado, salga del contenedor.

```cmd
exit
```

Ahora creará una nueva imagen de contenedor a partir del contenedor modificado. Para ver una lista de contenedores ejecute lo siguiente y anote el identificador de contenedor.

```console
docker ps -a
```

Ejecute el siguiente comando para crear la nueva imagen de "Hola a todos". Reemplace <containerid> con el identificador del contenedor.

```console
docker commit <containerid> helloworld
```

Una vez finalizado, tendrá una imagen personalizada que contiene el script de hola a todos. Esto puede verse con el comando siguiente.

```console
docker images
```

Finalmente, para quitar el contenedor, use el comando `docker run`.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

El resultado de la `docker run` comando es que se creó un contenedor de Hyper-V desde la imagen de "Hola a todos", se ha iniciado en el contenedor y ejecuta una lectura de nuestro archivo (salida Hola al shell) y, a continuación, el contenedor detiene y se quita una instancia de cmd.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtén información sobre cómo crear una aplicación de ejemplo](./building-sample-app.md)