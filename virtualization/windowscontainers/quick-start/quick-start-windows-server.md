---
title: Contenedores de Windows en Windows Server
description: Inicio rápido de implementación de contenedores
keywords: docker, contenedores
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: fd2de24a4c8c03817978a53b340e2a77285c69ad
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576716"
---
# <a name="windows-containers-on-windows-server"></a>Contenedores de Windows en Windows Server

En este ejercicio te guiará a través de la implementación básica y el uso de la característica de contenedor de Windows en Windows Server 2019 y Windows Server 2016.

En este inicio rápido llevará a cabo:

1. Habilitar la característica de contenedores en Windows Server
2. Instalar Docker
3. Ejecutar un contenedor simple de Windows

Si necesitas familiarizarte con los contenedores, encontrarás esta información en [Acerca de los contenedores](../about/index.md).

Este inicio rápido es específico de los contenedores de Windows Server en Windows Server 2019 y Windows Server 2016. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido, incluyendo contenedores en Windows 10.

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de que se cumplen los requisitos siguientes:
- Un equipo (físico o virtual) con Windows Server 2019. Si estás usando Windows Server 2019 Insider Preview, actualice a [Windows Server 2019 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019 ).

> Las actualizaciones críticas son necesarias para la característica Windows container funcione. Instale todas las actualizaciones antes de realizar los pasos que se indican en este tutorial.

Si quiere implementar en Azure, esta [plantilla](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template) le facilitará la tarea.

<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="install-docker"></a>Instalar Docker

Para instalar a Docker, usaremos el [módulo de PowerShell del proveedor OneGet](https://github.com/oneget/oneget) que trabaja con proveedores para realizar la instalación: en este caso la [MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider). El proveedor habilita la característica de los contenedores en la máquina. También tendrás que instalar Docker en él y tendrás que reiniciarlo. Para trabajar con contenedores de Windows es necesario Docker. Este consta de motor y de cliente.

Abre una sesión de PowerShell con privilegios elevados y ejecuta los comandos siguientes.

En primer lugar, instala el proveedor PackageManagement de Docker-Microsoft desde la [Galería de PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

A continuación, usa el módulo PackageManagement de PowerShell para instalar la versión más reciente de Docker.

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Cuando PowerShell te pregunte si se debe confiar en el origen del paquete "DockerDefault", escribe `A` para continuar con la instalación. Cuando la instalación se haya completado, reinicia el equipo.

```powershell
Restart-Computer -Force
```

> [!TIP]
> Si quieres actualizar Docker más adelante:
>  - Comprueba la versión instalada con `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Busca la versión actual con `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Cuando estés listo, actualiza con `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, seguido de `Start-Service Docker`

## <a name="install-windows-updates"></a>Instalar actualizaciones de Windows

Asegúrate de que el sistema de Windows Server está actualizado ejecutando:

```console
sconfig
```

Esto muestra un menú de configuración basado en texto, donde podrás elegir la opción 6 para descargar e instalar actualizaciones:

```console
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

## <a name="deploy-your-first-container"></a>Implementar el primer contenedor

Para este ejercicio, se descarga una imagen de ejemplo de .NET creada previamente desde el registro de Docker Hub y se implementa un contenedor simple que ejecuta una aplicación de .NET Hello World.  

Usa `docker run` para implementar el contenedor de .Net. También se descargará la imagen de contenedor, que puede tardar unos minutos. Según la versión de host de Windows Server, ejecute el siguiente comando siguiente.

#### <a name="windows-server-2019"></a>Windows Server 2019

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-1809
```

#### <a name="windows-server-2016"></a>Windows Server 2016

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-sac2016
```

El contenedor se iniciará, imprimirá el mensaje "hello world" y después se cerrará.

```console
         Hello from .NET Core!
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
Platform: .NET Core
OS: Microsoft Windows 10.0.17763
```

Para obtener información más detallada sobre el comando Run de Docker, consulta [Docker Run Reference (Referencia de Run de Docker)]( https://docs.docker.com/engine/reference/run/) en Docker.com.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtén información sobre cómo automatizar compilaciones del contenedor y guardar imágenes](./quick-start-images.md)
