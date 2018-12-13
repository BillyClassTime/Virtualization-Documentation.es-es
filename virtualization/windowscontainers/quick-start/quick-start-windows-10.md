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
ms.openlocfilehash: 4202908f8797a2b98ab657c45cd9a6b33191bd6f
ms.sourcegitcommit: 9dfef8d261f4650f47e8137a029e893ed4433a86
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/09/2018
ms.locfileid: "6224904"
---
# <a name="windows-and-linux-containers-on-windows-10"></a>Contenedores de Linux en Windows 10 y Windows

El ejercicio te guiará a través de la creación y ejecución de contenedores de Windows y Linux en Windows 10. Una vez finalizado, habrás de:

1. Instalado Docker para Windows
2. Ejecutar un contenedor simple de Windows
3. Ejecutar un contenedor simple de Linux con contenedores de Linux en Windows (LCOW)

Este inicio rápido es específico de Windows10. En la tabla de contenido en el lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

***Aislamiento de Hyper-V:*** Contenedores de Windows Server requieren aislamiento de Hyper-V en Windows 10 para ofrecer a los desarrolladores con la misma versión de kernel y configuración que se usará en producción, más acerca de Hyper-V aislamiento puede encontrarse en la página [contenedora acerca de Windows](../about/index.md) .

**Requisitos previos:**

- Un sistema de equipo físico que ejecuta Windows 10 Fall Creators Update (versión 1709) o posterior (Professional o Enterprise) que [puede ejecutar Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)

> Si no estás buscando para ejecutar LCOW en este tutorial, los contenedores de Windows podrán ejecutarse en Windows 10 Anniversary Update (versión 1607) o posterior.

## <a name="1-install-docker-for-windows"></a>1. Instalar Docker para Windows

[Descarga Docker para Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) y ejecuta el programa de instalación (se requiere inicio de sesión. Crear una cuenta si no tienes uno ya). Se encuentran disponibles [instrucciones de instalación detalladas](https://docs.docker.com/docker-for-windows/install) en la documentación de Docker (en inglés).

> Si ya tienes instalado de Docker, asegúrate de que tienes la versión 18.02 o posterior para admitir LCOW. Comprobar si ejecuta `docker -v` o comprobar *Acerca de Docker*.

> Opción de las características experimentales en *Docker configuración > Daemon* debe estar activado para ejecutar contenedores LCOW.

## <a name="2-switch-to-windows-containers"></a>2. Cambia a contenedores de Windows

Después de la instalación, Docker para Windows ejecuta contenedores de Linux de forma predeterminada. Cambia a contenedores de Windows mediante el menú de bandeja de Docker o ejecutando el siguiente comando en un símbolo del sistema de PowerShell: `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`.

![](./media/docker-for-win-switch.png)
> Ten en cuenta que el modo de contenedores de Windows permite para contenedores LCOW además de los contenedores de Windows.

## <a name="3-install-base-container-images"></a>3. Instala imágenes base del contenedor

Los contenedores de Windows se crean a partir de imágenes base. El comando siguientes extraerá la imagen base de Nano Server.

```
docker pull microsoft/nanoserver
```

Una vez extraída, si se ejecuta `docker images`, se devolverá una lista de las imágenes instaladas, en este caso la imagen de Nano Server.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Lee el CLUF de la imagen del sistema operativo de contenedores de Windows que se encuentra aquí: [CLUF](../images-eula.md).

## <a name="4-run-your-first-windows-container"></a>4. ejecuta el primer contenedor de Windows

En este ejemplo simple, se va a crear e implementar una imagen de contenedor "Hola a todos". Para lograr una mejor experiencia ejecute estos comandos en un shell de CMD de Windows con privilegios elevados o en PowerShell.
> Windows PowerShell ISE no funciona en sesiones interactivas con contenedores. Aunque el contenedor se ejecuta, parece que se cuelga.

En primer lugar, inicie un contenedor con una sesión interactiva desde la imagen `nanoserver`. Una vez que se ha iniciado el contenedor, aparecerá un shell de comandos desde dentro del mismo.  

```
docker run -it microsoft/nanoserver cmd
```

Dentro del contenedor se creará un script sencillo de "Hola a todos".

```
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

Una vez completado, salga del contenedor.

```
exit
```

Ahora creará una nueva imagen de contenedor a partir del contenedor modificado. Para ver una lista de contenedores ejecute lo siguiente y anote el identificador de contenedor.

```
docker ps -a
```

Ejecute el siguiente comando para crear la nueva imagen de "Hola a todos". Reemplace <containerid> con el identificador del contenedor.

```
docker commit <containerid> helloworld
```

Una vez finalizado, tendrá una imagen personalizada que contiene el script de hola a todos. Esto puede verse con el comando siguiente.

```
docker images
```

Finalmente, para quitar el contenedor, use el comando `docker run`.

```
docker run --rm helloworld powershell c:\helloworld.ps1
```

El resultado de este comando `docker run` es que se crea un contenedor de Hyper-V a partir de la imagen de "Hola a todos", a continuación se ejecuta un script "Hola a todos" de ejemplo (el resultado se muestra en el shell) y luego el contenedor se detiene y se quita.
Los siguientes inicios rápidos de Windows 10 y contenedores profundizarán en la creación e implementación de aplicaciones en contenedores en Windows 10.

## <a name="run-your-first-lcow-container"></a>Ejecutar el primer contenedor LCOW

Para este ejemplo, se implementará un contenedor BusyBox. En primer lugar, intentas ejecutar una imagen de "Hello World" BusyBox.

```
docker run --rm busybox echo hello_world
```

Ten en cuenta que esto devuelve un error cuando se intenta extraer la imagen de Docker. Esto ocurre porque Dockers requiere una directiva a través de la `--platform` marca para confirmar que el sistema operativo host y la imagen se comparan correctamente. Dado que la plataforma de forma predeterminada en el modo de contenedor de Windows es Windows, agrega un `--platform linux` marca para extraer y ejecutar el contenedor.

```
docker run --rm --platform linux busybox echo hello_world
```

Una vez que se ha extraído la imagen con la plataforma indica, el `--platform` marca ya no es necesario. Ejecuta el comando sin él para probarlo.

```
docker run --rm busybox echo hello_world
```

Ejecutar `docker images` para devolver una lista de imágenes instaladas. En este caso, las imágenes de Windows y Linux.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>Pasos siguientes

Extra: Consulta correspondiente de Docker [blog publicar](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) acerca de cómo ejecutar LCOW

Continúa en el siguiente tutorial para ver un ejemplo de [cómo crear una aplicación de ejemplo](./building-sample-app.md)
