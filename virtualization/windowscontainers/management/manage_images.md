---
title: Imágenes del contenedor de Windows
description: Cree y administre imágenes del contenedor con contenedores de Windows.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
---

# Imágenes del contenedor de Windows

**Esto es contenido preliminar y está sujeto a cambios.** 

Las imágenes de contenedor se utilizan para implementar contenedores. Estas imágenes pueden incluir un sistema operativo, aplicaciones y todas las dependencias de aplicaciones. Por ejemplo, puede desarrollar una imagen de contenedor que se haya configurado previamente con Nano Server, IIS y una aplicación que se ejecuta en IIS. Esta imagen del contenedor se puede almacenar después en un Registro del contenedor para su uso posterior, se puede implementar en cualquier host de contenedor de Windows (local, en la nube o incluso en un servicio de contenedor) y también se puede usar como base para una nueva imagen del contenedor.

Hay dos tipos de imágenes de contenedor:

**Imágenes del sistema operativo base**: las ofrece Microsoft e incluyen los componentes principales del sistema operativo. 

**Imágenes del contenedor**: son imágenes del contenedor personalizadas que se derivan de una imagen del sistema operativo base.

## Imágenes del sistema operativo base

### Instalar la imagen

Las imágenes del sistema operativo del contenedor se pueden encontrar e instalar con el módulo de PowerShell ContainerImage. Antes de utilizar este módulo, deberá instalarse. Se puede usar el siguiente comando para instalar el módulo. Para obtener más información sobre el uso del módulo OneGet de PowerShell de imagen del contenedor, vea [Container Image Provider](https://github.com/PowerShell/ContainerProvider) (Proveedor de imágenes de contenedores). 

```none
Install-PackageProvider ContainerImage -Force
```

Una vez instalado, se puede devolver una lista de imágenes del sistema operativo base usando `Find-ContainerImage`.

```none
Find-ContainerImage

Name                 Version          Source           Summary
----                 -------          ------           -------
NanoServer           10.0.14300.1010  ContainerImag... Container OS Image of Windows Server 2016 Technical...
WindowsServerCore    10.0.14300.1000  ContainerImag... Container OS Image of Windows Server 2016 Technical...
```

Para descargar e instalar la imagen del sistema operativo base Nano Server, ejecute lo siguiente. El parámetro `-version` es opcional. Sin una versión de imagen de SO base especificada, se instalará la versión más reciente.

```none
Install-ContainerImage -Name NanoServer -Version 10.0.14300.1010
```

Del mismo modo, este comando descargará e instalará la imagen del sistema operativo base de Windows Server Core. El parámetro `-version` es opcional. Sin una versión de imagen de SO base especificada, se instalará la versión más reciente.

```none
Install-ContainerImage -Name WindowsServerCore -Version 10.0.14300.1000
```

Compruebe que las imágenes se instalaron con el comando `docker images`. 

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     40356b90dc80        2 weeks ago         793.3 MB
windowsservercore   10.0.14304.1000     7837d9445187        2 weeks ago         9.176 GB
```  

Una vez instalado, podría interesarle etiquetar las imágenes con la etiqueta "latest". Estas instrucciones se detallan en la sección sobre el etiquetado que se encuentra a continuación.

> Si ha descargado la imagen del sistema operativo base pero no aparece cuando se ejecuta `docker images`, reinicie el servicio Docker mediante el applet del panel de control de servicios o el comando "sc stop docker" y, después, use "sc start docker".

### Etiquetar imágenes

Cuando se hace referencia a una imagen de contenedor mediante el nombre, el motor de Docker buscará la versión más reciente de la imagen. Si no se puede determinar la versión más reciente, se producirá el siguiente error.

```none
docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Una vez instaladas las imágenes del sistema operativo base de Windows Server Core o Nano Server, deberá etiquetarlas con una versión que indique “reciente”. Para ello, use el comando `docker tag`. 

Para obtener más información sobre `docker tag`, consulte [Tag, push, and pull your images (Etiquetar, insertar y extraer imágenes) en docker.com](https://docs.docker.com/mac/step_six/). 

```none
docker tag <image id> windowsservercore:latest
```

Cuando estén etiquetadas, la salida `docker images` mostrará dos versiones de la misma imagen: una con una etiqueta de la versión de la imagen y otra con una etiqueta "latest". Ahora puede hacer referencia a la imagen por su nombre.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14300.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### Instalación sin conexión

Las imágenes del sistema operativo base también pueden instalarse sin conexión a Internet. Para ello, descargue la imagen en un equipo con conexión a Internet, cópiela en el sistema de destino y, después, importe la imagen mediante el comando `Install-ContainerOSImages`.

Antes de descargar la imagen del sistema operativo base, ejecute el siguiente comando para preparar el sistema **conectado a Internet** con el proveedor de imágenes de contenedores.

```none
Install-PackageProvider ContainerImage -Force
```

Obtenga una lista de imágenes a partir del administrador de paquetes de PowerShell OneGet:

```none
Find-ContainerImage
```

Salida:

```none
Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.14300.1010         Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.14300.1000         Container OS Image of Windows Server 2016 Techn...
```

Para descargar una imagen, use el comando `Save-ContainerImage`.

```none
Save-ContainerImage -Name NanoServer -Path c:\container-image
```

Una vez hecho esto, podrá copiar la imagen de contenedor descargada en el **host de contenedor sin conexión** e instalarla mediante el comando `Install-ContainerOSImage`.

```none
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### Desinstalar una imagen del sistema operativo

Las imágenes del sistema operativo base pueden desinstalarse mediante el comando `Uninstall-ContainerOSImage`. En el ejemplo siguiente, se desinstalará la imagen del sistema operativo base NanoServer.

```none
Uninstall-ContainerOSImage -FullName CN=Microsoft_NanoServer_10.0.14304.1003
```

## Imágenes del contenedor

### Enumerar imágenes

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.14300.1000     6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.14300.1010     8572198a60f1        2 weeks ago          0 B
```

### Crear imagen

Puede crear una nueva imagen de contenedor a partir de un contenedor ya existente. Para ello, use el comando `docker commit`. En el siguiente ejemplo se crea una nueva imagen de contenedor con el nombre “windowsservercoreiis”.

```none
docker commit 475059caef8f windowsservercoreiis
```

### Quitar la imagen

Las imágenes del contenedor no se pueden quitar si cualquier contenedor, incluso en un estado detenido, tiene una dependencia de la imagen.

Cuando se quita una imagen con Docker, se puede hacer referencia a las imágenes por nombre de la imagen o identificador.

```none
docker rmi windowsservercoreiis
```

### Dependencia de imagen

Para ver las dependencias de la imagen con Docker, se puede usar el comando `docker history`.

```none
docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

El registro de Docker Hub contiene imágenes pregeneradas que se pueden descargar en un host de contenedor. Cuando estas imágenes se hayan descargado, pueden usarse como base para aplicaciones de contenedor de Windows.

Para ver una lista de imágenes disponibles en Docker Hub, use el comando `docker search`. Nota: Tendrá que instalar las imágenes del sistema operativo base Windows Server Core o Nano Server antes de extraer esas imágenes de contenedor dependientes de Docker Hub.

La mayoría de estas imágenes tiene una versión de Windows Server Core y una versión de Nano Server. Para obtener una versión específica, simplemente agregue la etiqueta ":windowsservercore" o ":nanoserver". La etiqueta "latest" devolverá la versión de Windows Server Core de forma predeterminada, a menos que solo haya disponible una versión de Nano Server.

> Las imágenes que comienzan con "nano-" dependen de la imagen del sistema operativo base Nano Server.

```none
docker search *

NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/sample-django  Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35       .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/sample-golang  Go Programming Language installed in a Win...   1                    [OK]
microsoft/sample-httpd   Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis            Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/sample-mongodb MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/sample-mysql   MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-nginx   Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-node    Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-python  Python installed in a Windows Server Core ...   1                    [OK]
microsoft/sample-rails   Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/sample-redis   Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-ruby    Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-sqlite  SQLite installed in a Windows Server Core ...   1                    [OK]
```

Para descargar una imagen de Docker Hub, use `docker pull`.

```none
docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

La imagen ahora estará visible cuando se ejecute `docker images`.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.14300.1000     6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```



<!--HONumber=May16_HO4-->


