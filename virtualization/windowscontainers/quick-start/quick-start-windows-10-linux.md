---
title: Contenedores de Windows y Linux en Windows 10
description: Inicio rápido de implementación de contenedores
keywords: Docker, contenedores, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909585"
---
# <a name="linux-containers-on-windows-10"></a>Contenedores de Linux en Windows 10

El ejercicio le guiará a través de la creación y ejecución de contenedores de Linux en Windows 10.

En esta guía de inicio rápido, realizará las siguientes acciones:

1. Instalación de Docker Desktop
2. Ejecución de un contenedor de Linux sencillo con contenedores de Linux en Windows (LCOW)

Este inicio rápido es específico de Windows 10. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de cumplir los siguientes requisitos:
- Un equipo físico con Windows 10 Professional, Windows 10 Enterprise o Windows Server 2019 versión 1809 o posterior
- Asegúrese [de que Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) está habilitado.

***Aislamiento de Hyper-V:*** Los contenedores de Linux en Windows requieren el aislamiento de Hyper-V en Windows 10 con el fin de proporcionar a los desarrolladores el kernel de Linux adecuado para ejecutar el contenedor. Encontrará más información sobre el aislamiento de Hyper-V en la página [acerca de los contenedores de Windows](../about/index.md) .

## <a name="install-docker-desktop"></a>Instalación de Docker Desktop

Descargue [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) y ejecute el instalador (se le pedirá que inicie sesión. Cree una cuenta si aún no tiene una. Se encuentran disponibles [instrucciones de instalación detalladas](https://docs.docker.com/docker-for-windows/install) en la documentación de Docker (en inglés).

> Si ya tiene Docker instalado, asegúrese de que tiene la versión 18,02 o posterior para admitir LCOW. Para realizar la comprobación, ejecute `docker -v` o consulte *acerca de Docker*.

> La opción ' características experimentales ' en la *configuración de docker > demonio* debe activarse para ejecutar contenedores de LCOW.

## <a name="run-your-first-lcow-container"></a>Ejecutar el primer contenedor de LCOW

En este ejemplo, se implementará un contenedor BusyBox. En primer lugar, intente ejecutar una imagen BusyBox ' Hola mundo '.

```console
docker run --rm busybox echo hello_world
```

Tenga en cuenta que esto devuelve un error cuando Docker intenta extraer la imagen. Esto se debe a que Dockers requiere una directiva a través de la marca `--platform` para confirmar que la imagen y el sistema operativo host coinciden adecuadamente. Dado que la plataforma predeterminada en el modo de contenedor de Windows es Windows, agregue una marca de `--platform linux` para extraer y ejecutar el contenedor.

```console
docker run --rm --platform linux busybox echo hello_world
```

Una vez que la imagen se ha extraído con la plataforma indicada, la marca de `--platform` ya no es necesaria. Ejecute el comando sin él para probarlo.

```console
docker run --rm busybox echo hello_world
```

Ejecute `docker images` para devolver una lista de imágenes instaladas. En este caso, las imágenes de Windows y Linux.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonus: vea la [entrada de blog](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) correspondiente de Docker en ejecutar LCOW.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtenga información sobre cómo compilar una aplicación de ejemplo](./building-sample-app.md)
