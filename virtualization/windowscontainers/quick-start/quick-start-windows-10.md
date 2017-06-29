---
title: Contenedor de Windows en Windows 10
description: "Inicio rápido de implementación de contenedores"
keywords: docker, contenedores
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a6ed0cccb984d303990973a1e2009cc2922f9443
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: es-ES
---
# <a name="windows-containers-on-windows-10"></a>Contenedores de Windows en Windows 10

Este ejercicio te guiará a través de la implementación básica y el uso de la característica de contenedor de Windows en Windows 10 Professional o Enterprise (Anniversary Edition). Una vez finalizado, habrás instalado Docker para Windows y ejecutarás un contenedor simple. Antes de comenzar este inicio rápido, familiarízate con la terminología y los conceptos básicos de los contenedores. Esta información se encuentra en la [Introducción a los contenedores](./index.md).

Esta introducción es específica para Windows 10. En la tabla de contenido del lado izquierdo de esta página encontrarás documentación adicional de inicio rápido.

**Requisitos previos:**

- Un sistema de equipo físico que ejecuta Windows 10 Anniversary Edition o Creators Update (Professional o Enterprise).   
- Este inicio rápido se puede ejecutar en una máquina virtual de Windows 10, pero tendrá que estar habilitada la virtualización anidada. Puedes encontrar más información en la [guía de virtualización anidada](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> Debe instalar las actualizaciones críticas para que funcionen los contenedores de Windows.
> Para comprobar la versión del SO, ejecute `winver.exe` y compare la versión que se muestra con el [Historial de actualizaciones de Windows 10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history).
> Asegúrese de tener la versión 14393.222 o posterior antes de continuar.

## <a name="1-install-docker-for-windows"></a>1. Instalar Docker para Windows

[Descarga Docker para Windows](https://download.docker.com/win/stable/InstallDocker.msi) y ejecuta el programa de instalación. Se encuentran disponibles [instrucciones de instalación detalladas](https://docs.docker.com/docker-for-windows/install) en la documentación de Docker (en inglés).

## <a name="2-switch-to-windows-containers"></a>2. Cambia a contenedores de Windows

Después de la instalación, Docker para Windows ejecuta contenedores de Linux de forma predeterminada. Cambia a contenedores de Windows mediante el menú de bandeja de Docker o ejecutando el siguiente comando en un símbolo del sistema de PowerShell: `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`.

![](./media/docker-for-win-switch.png)

## <a name="3-install-base-container-images"></a>3. Instala imágenes base del contenedor

Los contenedores de Windows se crean a partir de imágenes base. El comando siguientes extraerá la imagen base de Nano Server.

```none
docker pull microsoft/nanoserver
```

Una vez extraída, si se ejecuta `docker images`, se devolverá una lista de las imágenes instaladas, en este caso la imagen de Nano Server.

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Lee el CLUF de la imagen del sistema operativo de contenedores de Windows que se encuentra aquí: [CLUF](../images-eula.md).

## <a name="4-run-your-first-container"></a>4. Ejecuta el primer contenedor

En este ejemplo simple, se va a crear e implementar una imagen de contenedor "Hola a todos". Para lograr una mejor experiencia ejecute estos comandos en un shell de CMD de Windows con privilegios elevados o en PowerShell.

> Windows PowerShell ISE no funciona en sesiones interactivas con contenedores. Aunque el contenedor se ejecuta, parece que se cuelga.

En primer lugar, inicie un contenedor con una sesión interactiva desde la imagen `nanoserver`. Una vez que se ha iniciado el contenedor, aparecerá un shell de comandos desde dentro del mismo.  

```none
docker run -it microsoft/nanoserver cmd
```

Dentro del contenedor se creará un script sencillo de "Hola a todos".

```none
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

Una vez completado, salga del contenedor.

```none
exit
```

Ahora creará una nueva imagen de contenedor a partir del contenedor modificado. Para ver una lista de contenedores ejecute lo siguiente y anote el identificador de contenedor.

```none
docker ps -a
```

Ejecute el siguiente comando para crear la nueva imagen de "Hola a todos". Reemplace <containerid> con el identificador del contenedor.

```none
docker commit <containerid> helloworld
```

Una vez finalizado, tendrá una imagen personalizada que contiene el script de hola a todos. Esto puede verse con el comando siguiente.

```none
docker images
```

Finalmente, para quitar el contenedor, use el comando `docker run`.

```none
docker run --rm helloworld powershell c:\helloworld.ps1
```

El resultado de este comando `docker run` es que se crea un contenedor de Hyper-V a partir de la imagen de "Hola a todos", a continuación se ejecuta un script "Hola a todos" de ejemplo (el resultado se muestra en el shell) y luego el contenedor se detiene y se quita.
Los siguientes inicios rápidos de Windows 10 y contenedores profundizarán en la creación e implementación de aplicaciones en contenedores en Windows 10.

## <a name="next-steps"></a>Pasos siguientes

[Contenedores de Windows en Windows Server](./quick-start-windows-server.md)
