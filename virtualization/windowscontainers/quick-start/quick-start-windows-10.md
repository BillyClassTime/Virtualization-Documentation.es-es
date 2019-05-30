---
title: Contenedores de Windows y Linux en Windows 10
description: Inicio rápido de implementación de contenedores
keywords: acoplador, contenedores, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: ae311ecccdfbfc30b1079330a8eb02c1ce3ac94b
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680975"
---
# <a name="windows-containers-on-windows-10"></a>Contenedores de Windows en Windows 10

> [!div class="op_single_selector"]
> - [Contenedores de Linux en Windows](quick-start-windows-10-linux.md)
> - [Contenedores de Windows en Windows](quick-start-windows-10.md)

El ejercicio recorrerá la creación y la ejecución de contenedores de Windows en Windows 10.

En este inicio rápido, podrás realizar lo siguiente:

1. Instalar el escritorio del acoplador
2. Ejecutar un contenedor de Windows simple

Este inicio rápido es específico de Windows10. Puede encontrar documentación adicional de inicio rápido en la tabla de contenido, en la parte izquierda de esta página.

## <a name="prerequisites"></a>Requisitos previos
Asegúrate de cumplir los siguientes requisitos:
- Un sistema informático físico que ejecute Windows 10 Professional o Enterprise con la actualización de aniversario (versión 1607) o posterior. 
- Asegúrese [de que Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) está habilitado.

***Aislamiento de Hyper-V:*** Los contenedores de Windows Server requieren el aislamiento de Hyper-V en Windows 10 para proporcionar a los programadores la misma versión y configuración del núcleo que se usarán en la producción, encontrará más información sobre el aislamiento de Hyper-V en la página [acerca del contenedor de Windows](../about/index.md) .

> [!NOTE]
> En el lanzamiento de la actualización 2018 de octubre de Windows, ya no se permite que los usuarios ejecuten un contenedor de Windows en modo de aislamiento de procesos en Windows 10 Enterprise o Professional para fines de desarrollo y pruebas. Consulte las [preguntas más frecuentes](../about/faq.md) para obtener más información.

## <a name="install-docker-desktop"></a>Instalar el escritorio del acoplador

Descarga el [escritorio](https://store.docker.com/editions/community/docker-ce-desktop-windows) del acoplador y ejecuta el instalador (se te pedirá que inicies sesión. Crear una cuenta si aún no la tiene. Se encuentran disponibles [instrucciones de instalación detalladas](https://docs.docker.com/docker-for-windows/install) en la documentación de Docker (en inglés).

## <a name="switch-to-windows-containers"></a>Cambiar a contenedores de Windows

Después de instalar el acoplador de escritorio, se usa de forma predeterminada los contenedores de Linux. Cambie a contenedores de Windows mediante la bandeja del Dock-menú o ejecutando el siguiente comando en un símbolo del sistema de PowerShell:

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
> Lea el [CLUF](../images-eula.md)de la imagen del sistema de contenedores de Windows.

## <a name="run-your-first-windows-container"></a>Ejecutar el primer contenedor de Windows

Para este sencillo ejemplo, se creará e implementará una imagen de contenedor de "Hola a todos". Para lograr una mejor experiencia ejecute estos comandos en un shell de CMD de Windows con privilegios elevados o en PowerShell.

> Windows PowerShell ISE no funciona en sesiones interactivas con contenedores. Aunque el contenedor se ejecuta, parece que se cuelga.

En primer lugar, inicie un contenedor con una sesión interactiva desde la imagen `nanoserver`. Una vez que se ha iniciado el contenedor, aparecerá un shell de comandos desde dentro del mismo.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

Dentro del contenedor, crearemos un simple archivo de texto "Hola a todos".

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

El resultado del `docker run` comando es que un contenedor que se ejecuta con el aislamiento de Hyper-V se creó a partir de la imagen ' HelloWorld ', una instancia de CMD se inició en el contenedor y ejecutó una lectura de nuestro archivo (salida demostrada en el shell) y, a continuación, el contenedor detenido y eliminado.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre cómo crear una aplicación de ejemplo](./building-sample-app.md)
