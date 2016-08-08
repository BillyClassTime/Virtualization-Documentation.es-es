---
title: Contenedor de Windows en Windows 10
description: "Inicio rápido de implementación de contenedores"
keywords: docker, contenedores
author: neilpeterson
manager: timlt
ms.date: 07/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 6c7ce9f1767c6c6391cc6d33a553216bd815ff72
ms.openlocfilehash: bd93f5a73268b552710304d7da568e1497239679

---

# Contenedores de Windows en Windows 10

**Esto es contenido preliminar y está sujeto a cambios.** 

Este ejercicio le guiará a través de la implementación básica y el uso de la característica de contenedor de Windows en Windows 10 (compilación 14372 de Insider y superiores). Una vez realizado, habrá instalado el rol de contenedor e implementado un contenedor sencillo de Hyper-V. Antes de comenzar este inicio rápido, familiarícese con la terminología y los conceptos básicos de los contenedores. Esta información se encuentra en la [Introducción a los contenedores](./quick_start.md). 

Este inicio rápido es específico de los contenedores de Hyper-V en Windows 10. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

**Requisitos previos:**

- Un equipo físico con una [versión Insider de Windows 10](https://insider.windows.com/).   
- Este inicio rápido se puede ejecutar en una máquina virtual de Windows 10, pero la virtualización anidada tendrá que estar habilitada. Puede encontrar más información en la [guía de virtualización anidada](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

## 1. Instalar la característica de contenedor

La característica de contenedor debe habilitarse antes de trabajar con contenedores de Windows. Para ello, ejecute el comando siguiente en una sesión de PowerShell con privilegios elevados. 

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

Una vez realizada la copia de seguridad, ejecute el comando siguiente para solucionar un problema conocido con la vista previa técnica de contenedores de Windows.  

 ```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

## 2. Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. En este ejercicio se instalarán ambos. Para ello, ejecute el siguiente comando. 

Descargue el motor de Docker y el cliente como un archivo zip.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Expanda el archivo zip en Archivos de programa. El contenido del archivo ya está en el directorio de Docker.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Agregue el directorio de Docker a la ruta de acceso del sistema.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
```

Reinicie la sesión de PowerShell para que reconozca la ruta de acceso modificada.

Para instalar Docker como un servicio de Windows, ejecute lo siguiente.

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
```

Una vez instalado, puede iniciar el servicio.

```none
Start-Service Docker
```

## 3. Instalar imágenes base del contenedor

Los contenedores de Windows se implementan a partir de plantillas o imágenes. Para poder implementar un contenedor, es necesario descargar una imagen base del sistema operativo del contenedor. Los comandos siguientes descargarán la imagen base de Nano Server.
    
> Este procedimiento se aplica a las compilaciones de Windows Insiders superiores a la 14372 y es temporal hasta que 'docker pull' esté funcional.

Descargue la imagen base de Nano Server. 

```none
Start-BitsTransfer https://aka.ms/tp5/6b/docker/nanoserver -Destination nanoserver.tar.gz
```

Instale la imagen base.

```none  
docker load -i nanoserver.tar.gz
```

En este punto, si se ejecuta `docker images`, se devolverá una lista de imágenes instaladas, en este caso la imagen de Nano Server.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
```

Antes de continuar, debe etiquetar la imagen con una versión "reciente". Para ello, ejecute el comando siguiente.

```none
docker tag microsoft/nanoserver:10.0.14300.1030 nanoserver:latest
```

Para obtener información detallada sobre las imágenes de contenedor de Windows, consulte [Administración de imágenes del contenedor](../management/manage_images.md).

## 4. Implementar el primer contenedor

En este ejemplo simple, se va a crear e implementar una imagen de contenedor "Hola a todos". Para una mejor experiencia ejecute estos comandos en un shell de CMD de Windows con privilegios elevados.

En primer lugar, inicie un contenedor con una sesión interactiva desde la imagen `nanoserver`. Una vez que se ha iniciado el contenedor, aparecerá un shell de comandos desde dentro del mismo.  

```none
docker run -it nanoserver cmd
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

El resultado de este comando `docker run` es que se crea un contenedor de Hyper-V a partir de la imagen de "Hola a todos", a continuación se ejecuta un script "Hola a todos" de ejemplo (el resultado se muestra en el shell) y luego el contenedor se detiene y se quita. Los siguientes inicios rápidos de Windows 10 y contenedores profundizarán en la creación e implementación de aplicaciones en contenedores en Windows 10.

## Pasos siguientes

[Contenedores de Windows en Windows Server](./quick_start_windows_server.md)





<!--HONumber=Aug16_HO1-->


