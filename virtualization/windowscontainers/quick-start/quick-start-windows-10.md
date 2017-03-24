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
translationtype: Human Translation
ms.sourcegitcommit: 996d3b1a8f7c8325ac66d331e1d62208c0cf6b53
ms.openlocfilehash: 091a3570291624a3be40e3aabb9f99a482cb6470
ms.lasthandoff: 02/27/2017

---

# Contenedores de Windows en Windows 10

Este ejercicio le guiará a través de la implementación básica y el uso de la característica de contenedor de Windows en Windows 10 Professional o Enterprise (Anniversary Edition). Una vez realizado, habrá instalado el rol de contenedor e implementado un contenedor sencillo de Hyper-V. Antes de comenzar este inicio rápido, familiarícese con la terminología y los conceptos básicos de los contenedores. Esta información se encuentra en la [Introducción al inicio rápido](./index.md).

Este inicio rápido es específico de los contenedores de Hyper-V en Windows 10. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

**Requisitos previos:**

- Un sistema de equipo físico que ejecuta Windows 10 Anniversary Edition (Professional o Enterprise).   
- Este inicio rápido se puede ejecutar en una máquina virtual de Windows 10, pero la virtualización anidada tendrá que estar habilitada. Puede encontrar más información en la [guía de virtualización anidada](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> Debe instalar las actualizaciones críticas para que funcionen los contenedores de Windows. 
> Para comprobar la versión del SO, ejecute `winver.exe` y compare la versión que se muestra con el [Historial de actualizaciones de Windows 10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history). 
> Asegúrese de tener la versión 14393.222 o una posterior antes de continuar.

## 1. Instalar la característica de contenedor

La característica de contenedor debe habilitarse antes de trabajar con contenedores de Windows. Para ello, ejecute el comando siguiente en una sesión de PowerShell **con privilegios elevados**.

Si recibe un error que dice que `Enable-WindowsOptionalFeature` no existe, compruebe que está ejecutando PowerShell como administrador.

```none
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

Dado que Windows 10 solo admite contenedores de Hyper-V, la característica de Hyper-V también debe estar habilitada. Para habilitar la característica de Hyper-V mediante PowerShell, ejecute el siguiente comando en una sesión de PowerShell con privilegios elevados.

```none
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

Cuando la instalación haya finalizado, reinicie el equipo.

```none
Restart-Computer -Force
```

> Si antes usaba contenedores de Hyper-V en Windows 10 con las imágenes base del contenedor de Technical Preview 5, asegúrese de volver a habilitar OpLocks. Ejecute el comando siguiente:  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2. Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. En este ejercicio se instalarán ambos. Para ello, ejecute el siguiente comando.

Descargue el motor de Docker y el cliente como un archivo zip.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-17.03.0-ce.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Expanda el archivo zip en Archivos de programa. El contenido del archivo ya está en el directorio de Docker.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Agregue el directorio de Docker a la ruta de acceso del sistema.

```none
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

Para instalar Docker como un servicio de Windows, ejecute lo siguiente.

```none
dockerd --register-service
```

Una vez instalado, puede iniciar el servicio.

```none
Start-Service Docker
```

## 3. Instalar imágenes base del contenedor

Los contenedores de Windows se implementan a partir de plantillas o imágenes. Para poder implementar un contenedor, es necesario descargar una imagen base del sistema operativo del contenedor. Los comandos siguientes descargarán la imagen base de Nano Server.

Extraiga la imagen base de Nano Server.

```none
docker pull microsoft/nanoserver
```

Una vez extraída, si se ejecuta `docker images`, se devolverá una lista de imágenes instaladas, en este caso la imagen de Nano Server.

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Lee el CLUF de la imagen de sistema operativo de contenedores de Windows que se encuentra aquí: [CLUF](../images-eula.md).

## 4. Implementar el primer contenedor

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

## Pasos siguientes

[Contenedores de Windows en Windows Server](./quick-start-windows-server.md)

