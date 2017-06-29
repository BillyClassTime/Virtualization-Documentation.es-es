---
title: Contenedores de Windows en Windows Server
description: "Inicio rápido de implementación de contenedores"
keywords: docker, contenedores
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: be51a89292f339c40b05bb48e0921777fd4a9801
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: es-ES
---
# <a name="windows-containers-on-windows-server"></a>Contenedores de Windows en Windows Server

Este ejercicio te guiará a través de la implementación básica y el uso de la característica de contenedores de Windows en Windows Server 2016. Durante este ejercicio, instalarás el rol de contenedor e implementarás un contenedor simple de Windows Server. Antes de comenzar este inicio rápido, familiarízate con la terminología y los conceptos básicos de los contenedores. Podrás encontrar esta información en la [Introducción a los contenedores](./index.md).

Este inicio rápido es específico de los contenedores de Windows Server en Windows Server 2016. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido, incluyendo contenedores en Windows 10.

**Requisitos previos:**

Un equipo (físico o virtual) con Windows Server 2016. Si usa Windows Server 2016 TP5, actualice a [Windows Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

> Las actualizaciones críticas son necesarias para que la característica Windows Container funcione. Instale todas las actualizaciones antes de realizar los pasos que se indican en este tutorial.

Si quiere implementar en Azure, esta [plantilla](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template) le facilitará la tarea.<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="1-install-docker"></a>1. Instalar Docker

Para instalar Docker, usaremos el [módulo de PowerShell de proveedor OneGet](https://github.com/oneget/oneget), que trabaja con proveedores para realizar la instalación, en este caso [MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider). El proveedor habilita la característica de los contenedores en la máquina. También tendrás que instalar Docker en él y tendrás que reiniciarlo. Para trabajar con contenedores de Windows es necesario Docker. Este consta de motor y de cliente.

Abre una sesión de PowerShell con privilegios elevados y ejecuta los comandos siguientes.

En primer lugar, instala el proveedor PackageManagement de Docker-Microsoft desde la [Galería de PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

A continuación, usa el módulo PackageManagement de PowerShell para instalar la versión más reciente de Docker.
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Cuando PowerShell te pregunte si se debe confiar en el origen del paquete "DockerDefault", escribe `A` para continuar con la instalación. Cuando la instalación se haya completado, reinicia el equipo.

```none
Restart-Computer -Force
```

> Sugerencia: Si quieres actualizar Docker más adelante:
>  - Comprueba la versión instalada con `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Busca la versión actual con `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Cuando estés listo, actualiza con `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, seguido de `Start-Service Docker`

## <a name="2-install-windows-updates"></a>2. Instala las actualizaciones de Windows

Asegúrate de que el sistema de Windows Server está actualizado ejecutando:

```none
sconfig
```

Esto muestra un menú de configuración basado en texto, donde podrás elegir la opción 6 para descargar e instalar actualizaciones:

```none
===============================================================================
                         Server Configuration
===============================================================================

1) Domain/Workgroup:                    Workgroup:  WORKGROUP
2) Computer Name:                       WIN-HEFDK4V68M5
3) Add Local Administrator
4) Configure Remote Management          Enabled

5) Windows Update Settings:             DownloadOnly
6) Download and Install Updates
7) Remote Desktop:                      Disabled
...
```

Cuando se te solicite, elija la opción A para descargar todas las actualizaciones.

## <a name="3-deploy-your-first-container"></a>3. Implementar el primer contenedor

Para este ejercicio, se descarga una imagen de ejemplo de .NET creada previamente desde el registro de Docker Hub y se implementa un contenedor simple que ejecuta una aplicación de .NET Hello World.  

Usa `docker run` para implementar el contenedor de .Net. También se descargará la imagen de contenedor, que puede tardar unos minutos.

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver
```

El contenedor se iniciará, imprimirá el mensaje "hello world" y después se cerrará.

```console
         Dotnet-bot: Welcome to using .NET Core!
    __________________
                      \
                       \
                          ....
                          ....'
                           ....
                        ..........
                    .............'..'..
                 ................'..'.....
               .......'..........'..'..'....
              ........'..........'..'..'.....
             .'....'..'..........'..'.......'.
             .'..................'...   ......
             .  ......'.........         .....
             .                           ......
            ..    .            ..        ......
           ....       .                 .......
           ......  .......          ............
            ................  ......................
            ........................'................
           ......................'..'......    .......
        .........................'..'.....       .......
     ........    ..'.............'..'....      ..........
   ..'..'...      ...............'.......      ..........
  ...'......     ...... ..........  ......         .......
 ...........   .......              ........        ......
.......        '...'.'.              '.'.'.'         ....
.......       .....'..               ..'.....
   ..       ..........               ..'........
          ............               ..............
         .............               '..............
        ...........'..              .'.'............
       ...............              .'.'.............
      .............'..               ..'..'...........
      ...............                 .'..............
       .........                        ..............
        .....


**Environment**
Platform: .NET Core 1.0
OS: Microsoft Windows 10.0.14393
```

Para obtener información más detallada sobre el comando Run de Docker, consulta [Docker Run Reference (Referencia de Run de Docker)]( https://docs.docker.com/engine/reference/run/) en Docker.com.

## <a name="next-steps"></a>Pasos siguientes

[Imágenes de contenedores en Windows Server](./quick-start-images.md)

[Contenedores de Windows en Windows 10](./quick-start-windows-10.md)
