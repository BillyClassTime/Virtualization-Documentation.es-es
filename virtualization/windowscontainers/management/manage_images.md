---
title: "Imágenes del contenedor de Windows"
description: "Cree y administre imágenes del contenedor con contenedores de Windows."
keywords: docker, contenedores
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
redirect_url: https://docs.docker.com/v1.8/userguide/dockerimages/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 49e29949fc91533cf1d3ed0aef47e829276772f4

---

# Imágenes del contenedor de Windows

**Esto es contenido preliminar y está sujeto a cambios.** 

>Los contenedores de Windows se administran con Docker. La documentación sobre los contenedores de Windows es complementaria a la que se puede encontrar en [docs.docker.com](https://docs.docker.com/).

Las imágenes de contenedor se utilizan para implementar contenedores. Estas imágenes pueden incluir aplicaciones y todas las dependencias de aplicaciones. Por ejemplo, puede desarrollar una imagen de contenedor que se haya configurado previamente con Nano Server, IIS y una aplicación que se ejecuta en IIS. Esta imagen del contenedor se puede almacenar después en un Registro del contenedor para su uso posterior, se puede implementar en cualquier host de contenedor de Windows (local, en la nube o incluso en un servicio de contenedor) y también se puede usar como base para una nueva imagen del contenedor.

### Instalar la imagen

Antes de trabajar con contenedores de Windows, debe instalarse una imagen base. Las imágenes base están disponibles con Windows Server Core o Nano Server como el sistema operativo subyacente. Para obtener información sobre las configuraciones compatibles, consulte [Requisitos del sistema del contenedor de Windows](../deployment/system_requirements.md).

Para instalar la imagen base de Windows Server Core, ejecute lo siguiente:

```none
docker pull microsoft/windowsservercore
```

Para instalar la imagen base de Nano Server, ejecute lo siguiente:

```none
docker pull microsoft/nanoserver
```

### Enumerar imágenes

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        9 weeks ago         7.764 GB
microsoft/nanoserver          latest              3a703c6e97a2        9 weeks ago         969.8 MB
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

### Extracción de Docker

Para descargar una imagen de Docker Hub, use `docker pull`. Para más información, consulte [Docker Pull en Docker.com](https://docs.docker.com/engine/reference/commandline/pull/).

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

> Si se produce un error en la extracción de Docker, asegúrese de que se han aplicado las últimas actualizaciones acumulativas al host del contenedor. Puede encontrar en la actualización TP5 en [KB3157663]( https://support.microsoft.com/en-us/kb/3157663).

### Inserción de Docker

También es posible cargar imágenes en Docker Hub o un registro de confianza de Docker. Cuando se han cargado estas imágenes, se pueden descargar y volver a utilizar en distintos entornos de contenedor de Windows.

Para cargar una imagen de contenedor en Docker Hub, primero inicie sesión en el registro. Para más información, consulte [Docker Login en Docker.com]( https://docs.docker.com/engine/reference/commandline/login/).

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: username
Password:

Login Succeeded
```

Una vez iniciada sesión en Docker Hub o en el registro de confianza de Docker, utilice `docker push` para cargar una imagen de contenedor. Puede hacer referencia a la imagen de contenedor por nombre o identificador. Para más información, consulte [Docker Push en Docker.com]( https://docs.docker.com/engine/reference/commandline/push/).

```none
docker push username/containername

The push refers to a repository [docker.io/username/containername]
b567cea5d325: Pushed
00f57025c723: Pushed
2e05e94480e9: Pushed
63f3aa135163: Pushed
469f4bf35316: Pushed
2946c9dcfc7d: Pushed
7bfd967a5e43: Pushed
f64ea92aaebc: Pushed
4341be770beb: Pushed
fed398573696: Pushed
latest: digest: sha256:ae3a2971628c04d5df32c3bbbfc87c477bb814d5e73e2787900da13228676c4f size: 2410
```

En este momento la imagen del contenedor está disponible y puede obtenerse con `docker pull`.






<!--HONumber=Sep16_HO2-->


