---
title: Contenedores de Windows en Windows Server
description: "Inicio rápido de implementación de contenedores"
keywords: docker, contenedores
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: d4b509054aebf510b650ec21ede4df2515a87827
ms.openlocfilehash: 6e2232c8a043d482c0d6b734a66762c2d22003fe

---

# Contenedores de Windows en Windows Server

**Esto es contenido preliminar y está sujeto a cambios.**

Este ejercicio le guiará a través de la implementación básica y el uso de la característica de contenedores de Windows en Windows Server. Una vez realizado, habrá instalado el rol de contenedor e implementado un contenedor sencillo de Windows Server. Antes de comenzar este inicio rápido, familiarícese con la terminología y los conceptos básicos de los contenedores. Esta información se encuentra en la [Introducción a los contenedores](./quick_start.md).

Este inicio rápido es específico de los contenedores de Windows Server en Windows Server 2016. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

**Requisitos previos:**

Un equipo (físico o virtual) con [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview).

Hay una imagen de Windows Server totalmente configurada disponible en Azure. Para usar esta imagen, implemente una máquina virtual haciendo clic en el botón siguiente. La implementación tardará unos 10 minutos. Una vez completada, inicie sesión en la máquina virtual de Azure y vaya al paso 4 de este tutorial. 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

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

Descargue el motor de Docker y el cliente como un archivo zip.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.1.zip" -OutFile "$env:TEMP\docker-1.12.1.zip" -UseBasicParsing
```

Expanda el archivo zip en Archivos de programa.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.1.zip" -DestinationPath $env:ProgramFiles
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
dockerd --register-service
```

Una vez instalado, puede iniciar el servicio.

```none
Start-Service docker
```

## 3. Instalar imágenes base del contenedor

Los contenedores de Windows se implementan a partir de plantillas o imágenes. Para implementar un contenedor, es necesario descargar una imagen base del sistema operativo. El comando siguiente descargará la imagen base de Windows Server Core.

```none
docker pull microsoft/windowsservercore
```

Como este proceso puede tardar algún tiempo, puede dedicarse a otros asuntos y retomarlo cuando se haya completado la extracción.

Una vez extraída, si se ejecuta `docker images`, se devolverá una lista de imágenes instaladas, en este caso la imagen de Windows Server Core.

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        8 weeks ago         7.764 GB
```

Para obtener información detallada sobre las imágenes de contenedor de Windows, consulte [Administración de imágenes del contenedor](../management/manage_images.md).

## 4. Implementar el primer contenedor

Para este ejercicio, se descarga una imagen de IIS creada previamente desde el Registro de Docker Hub y se implementa un contenedor sencillo con IIS.  

Para buscar imágenes de contenedores de Windows en Docker Hub, ejecute `docker search Microsoft`.  

```none
docker search microsoft

NAME                                         DESCRIPTION
microsoft/aspnet                             ASP.NET is an open source server-side Web ...
microsoft/dotnet                             Official images for working with .NET Core...
mono                                         Mono is an open source implementation of M...
microsoft/azure-cli                          Docker image for Microsoft Azure Command L...
microsoft/iis                                Internet Information Services (IIS) instal...
microsoft/mssql-server-2014-express-windows  Microsoft SQL Server 2014 Express installe...
microsoft/nanoserver                         Nano Server base OS image for Windows cont...
microsoft/windowsservercore                  Windows Server Core base OS image for Wind...
microsoft/oms                                Monitor your containers using the Operatio...
microsoft/dotnet-preview                     Preview bits for microsoft/dotnet image
microsoft/dotnet35
microsoft/applicationinsights                Application Insights for Docker helps you ...
microsoft/sample-redis                       Redis installed in Windows Server Core and...
microsoft/sample-node                        Node installed in a Nano Server based cont...
microsoft/sample-nginx                       Nginx installed in Windows Server Core and...
microsoft/sample-httpd                       Apache httpd installed in Windows Server C...
microsoft/sample-dotnet                      .NET Core running in a Nano Server container
microsoft/sqlite                             SQLite installed in a Windows Server Core ...
...
```

Descargue la imagen de IIS mediante `docker pull`.  

```none
docker pull microsoft/iis
```

La descarga de la imagen puede comprobarse con el comando `docker images`. Observe que aparecerá tanto la imagen base (windowsservercore) como la imagen de IIS.

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis                 latest              accd044753c1        11 days ago         7.907 GB
microsoft/windowsservercore   latest              02cb7f65d61b        8 weeks ago         7.764 GB
```

Usuario `docker run` para implementar el contenedor de IIS.

```none
docker run -d -p 80:80 microsoft/iis ping -t localhost
```

Este comando ejecuta la imagen de IIS como un servicio en segundo plano (-d) y configura la red de forma que el puerto 80 del host de contenedor se asigne al puerto 80 del contenedor.
Para obtener información más detallada sobre el comando Run de Docker, consulte [Docker Run Reference (Referencia de Run de Docker)]( https://docs.docker.com/engine/reference/run/) en Docker.com.


Los contenedores en ejecución pueden verse con el comando `docker ps`. Tome nota del nombre del contenedor, ya que se usará en un paso posterior.

```none
docker ps

CONTAINER ID  IMAGE          COMMAND              CREATED             STATUS             PORTS               NAME
09c9cc6e4f83  microsoft/iis  "ping -t localhost"  About a minute ago  Up About a minute  0.0.0.0:80->80/tcp  big_jang
```

Desde otro equipo, abra un explorador web y escriba la dirección IP del host de contenedor. Si todo se ha configurado correctamente, debería ver la pantalla de presentación de IIS. Se sirve desde la instancia de IIS hospedada en el contenedor de Windows.

**Nota**: Si está trabajando en Azure, necesitará la dirección IP externa de la máquina virtual y una seguridad de red configurada. Para más información, consulte [Cómo administrar grupos de seguridad de red con el Portal de Azure]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg).

![](media/iis1.png)

De vuelta en el host de contenedor, use el comando `docker rm` para quitar el contenedor. Nota: Reemplace el nombre del contenedor de este ejemplo por el nombre real del contenedor.

```none
docker rm -f big_jang
```
## Pasos siguientes

[Imágenes de contenedores en Windows Server](./quick_start_images.md)

[Contenedores de Windows en Windows 10](./quick_start_windows_10.md)



<!--HONumber=Sep16_HO3-->


