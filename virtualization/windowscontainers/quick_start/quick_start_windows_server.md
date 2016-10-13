---
title: Contenedores de Windows en Windows Server
description: "Inicio rápido de implementación de contenedores"
keywords: docker, contenedores
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: ac962391cd3b82be2dd18b145ee5e6d7a483a91a
ms.openlocfilehash: 334f19fa645ad50eb59ad61890842f0b6a43dce2

---

# Contenedores de Windows en Windows Server

Este ejercicio le guiará a través de la implementación básica y el uso de la característica de contenedores de Windows en Windows Server. Una vez realizado, habrá instalado el rol de contenedor e implementado un contenedor sencillo de Windows Server. Antes de comenzar este inicio rápido, familiarícese con la terminología y los conceptos básicos de los contenedores. Esta información se encuentra en la [Introducción a los contenedores](./quick_start.md).

Este inicio rápido es específico de los contenedores de Windows Server en Windows Server 2016. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

**Requisitos previos:**

Un equipo (físico o virtual) con Windows Server 2016. Si usa Windows Server 2016 TP5, actualice a [Windows Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

> Las actualizaciones críticas son necesarias para que la característica Windows Container funcione. Instale todas las actualizaciones antes de realizar los pasos que se indican en este tutorial.

## 1. Instalar la característica de contenedor

La característica de contenedor debe habilitarse antes de trabajar con contenedores de Windows. Para ello, ejecute el comando siguiente en una sesión de PowerShell con privilegios elevados.

```none
Install-WindowsFeature containers
```

Cuando la instalación de la característica haya finalizado, reinicie el equipo.

```none
Restart-Computer -Force
```

## 2. Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. En este ejercicio se instalarán ambos.

Descargue la versión candidata para lanzamiento del cliente Commercially Supported Docker Engine como un archivo zip.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker-1.12.2.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Expanda el archivo zip en Archivos de programa.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Agregue el directorio de Docker a la ruta de acceso del sistema.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Para instalar Docker como un servicio de Windows, ejecute lo siguiente.

```none
dockerd.exe --register-service
```

Una vez instalado, puede iniciar el servicio.

```none
Start-Service docker
```

## 3. Implementar el primer contenedor

Para este ejercicio, se descarga una imagen de ejemplo de .NET creada previamente desde el registro de Docker Hub y se implementa un contenedor simple que ejecuta una aplicación de .NET Hello World.  

Use `docker run` para implementar el contenedor de .NET. También se descargará la imagen de contenedor, que puede tardar unos minutos.

```none
docker run microsoft/sample-dotnet
```

El contenedor se iniciará, imprimirá el mensaje hello world y después se cerrará.

```none
       Welcome to .NET Core!
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
```

Para obtener información más detallada sobre el comando Run de Docker, consulte [Docker Run Reference (Referencia de Run de Docker)]( https://docs.docker.com/engine/reference/run/) en Docker.com.

## Pasos siguientes

[Imágenes de contenedores en Windows Server](./quick_start_images.md)

[Contenedores de Windows en Windows 10](./quick_start_windows_10.md)



<!--HONumber=Oct16_HO2-->


