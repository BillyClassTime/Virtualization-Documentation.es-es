---
title: Contenedor de Windows en Windows 10
description: Inicio rápido de implementación de contenedores
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
---

# Contenedores de Windows en Windows 10

**Esto es contenido preliminar y está sujeto a cambios.** 

Este ejercicio le guiará a través de la implementación básica y el uso de la característica de contenedor de Windows en Windows 10 (compilación de Insider 14352 y superiores). Una vez realizado, habrá instalado el rol de contenedor e implementado un contenedor sencillo de Hyper-V. Antes de comenzar este inicio rápido, familiarícese con la terminología y los conceptos básicos de los contenedores. Esta información se encuentra en la [Introducción a los contenedores](./quick_start.md). 

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

## 2. Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. En este ejercicio se instalarán ambos. Para ello, ejecute el siguiente comando. 

Cree una carpeta para los ejecutables de Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Descargue el demonio de Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Descargue el cliente de Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Agregue el directorio de Docker a la ruta de acceso del sistema.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Reinicie la sesión de PowerShell para que reconozca la ruta de acceso modificada.

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
    
Establezca la directiva de ejecución de PowerShell del proceso actual de PowerShell. Esto solo afectará a los scripts que se ejecuten en la sesión actual de PowerShell, de todas formas, se deben tomar precauciones al cambiar la directiva de ejecución.

```none
Set-ExecutionPolicy Bypass -scope Process
```

Instale el proveedor de paquetes de imágenes del contenedor.

```none  
Install-PackageProvider ContainerImage -Force
```

Luego instale la imagen de Nano Server.

```none
Install-ContainerImage -Name NanoServer
```

Cuando se haya instalado la imagen base, debe reiniciar el servicio Docker.

```none
Restart-Service docker
```

En este punto, si se ejecuta `docker images`, se devolverá una lista de imágenes instaladas, en este caso la imagen de Nano Server.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
```

Antes de continuar, debe etiquetar la imagen con una versión "reciente". Para ello, ejecute el comando siguiente.

```none
docker tag nanoserver:10.0.14300.1016 nanoserver:latest
```

Para obtener información detallada sobre las imágenes de contenedor de Windows, consulte [Administración de imágenes del contenedor](../management/manage_images.md).

## 4. Implementar el primer contenedor

En este ejemplo sencillo, se ha creado previamente una imagen de .NET Core. Descargue esta imagen mediante el comando `docker pull`.

Cuando se ejecute, se iniciará un contenedor, se ejecutará la aplicación .NET Core sencilla y luego se cerrará el contenedor. 

```none
docker pull microsoft/sample-dotnet
```

Esto se puede comprobar con el comando `docker images`.

```none
docker images

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
microsoft/sample-dotnet  latest              28da49c3bff4        41 hours ago        918.3 MB
nanoserver               10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
nanoserver               latest              3f5112ddd185        3 weeks ago         810.2 MB
```

Ejecute el contenedor con el comando `docker run`. En el ejemplo siguiente se especifica el parámetro `--rm`, que indica al motor de Docker que elimine el contenedor cuando ya no se esté ejecutando. 

Para obtener información más detallada sobre el comando Run de Docker, consulte [Docker Run Reference (Referencia de Run de Docker)]( https://docs.docker.com/engine/reference/run/) en Docker.com.

```none
docker run --isolation=hyperv --rm microsoft/sample-dotnet
```

El resultado de este comando es que se crea un contenedor de Hyper-V a partir de la imagen sample-dotnet, entonces se ejecuta una aplicación de ejemplo (el resultado se muestra en el shell) y luego el contenedor se detiene y se quita. Los siguientes inicios rápidos de Windows 10 y contenedores profundizarán en la creación e implementación de aplicaciones en contenedores en Windows 10.

## Pasos siguientes

[Contenedores de Windows en Windows Server](./quick_start_windows_server.md)




<!--HONumber=Jun16_HO2-->


