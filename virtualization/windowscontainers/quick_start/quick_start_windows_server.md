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
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 80a2efcb9fff58d357dc1630a335665fc91147cb

---

# Contenedores de Windows en Windows Server

Este ejercicio le guiará a través de la implementación básica y el uso de la característica de contenedores de Windows en Windows Server. Una vez realizado, habrá instalado el rol de contenedor e implementado un contenedor sencillo de Windows Server. Antes de comenzar este inicio rápido, familiarícese con la terminología y los conceptos básicos de los contenedores. Esta información se encuentra en la [Introducción a los contenedores](./quick_start.md).

Este inicio rápido es específico de los contenedores de Windows Server en Windows Server 2016. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

**Requisitos previos:**

Un equipo (físico o virtual) con Windows Server 2016. Si usa Windows Server 2016 TP5, actualice a [Windows Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

> Las actualizaciones críticas son necesarias para que la característica Windows Container funcione. Instale todas las actualizaciones antes de realizar los pasos que se indican en este tutorial.

## 1. Instalar Docker

Para instalar Docker, usaremos el [módulo de PowerShell del proveedor OneGet](https://github.com/oneget/oneget). El proveedor habilitará la característica de contenedores en la máquina e instalará Docker, lo que requerirá un reinicio. Para trabajar con contenedores de Windows es necesario Docker. Este consta de motor y de cliente.

Abra una sesión de PowerShell con privilegios elevados y ejecute los comandos siguientes.

Primero instalaremos el módulo de PowerShell de OneGet.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Después, usaremos OneGet para instalar la versión más reciente de Docker.
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Cuando PowerShell le pregunte si se debe confiar en el origen del paquete "DockerDefault", escriba A para continuar con la instalación. Cuando finalice la instalación, reinicie el equipo.

```none
Restart-Computer -Force
```

## 2. Instalar actualizaciones de Windows

Para comprobar que su sistema de Windows Server esté actualizado, le recomendamos que instale actualizaciones de Windows ejecutando el siguiente comando:

```none
sconfig
```

Verá un menú de configuración basado en texto, donde puede elegir la opción 6 para descargar e instalar actualizaciones:

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

Cuando se le solicite, elija la opción A descargar todas las actualizaciones.

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



<!--HONumber=Oct16_HO4-->


