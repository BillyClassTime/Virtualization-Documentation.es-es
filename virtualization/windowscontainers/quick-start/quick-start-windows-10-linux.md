---
title: Contenedores de Linux en Windows 10 y Windows
description: Inicio rápido de implementación de contenedores
keywords: docker, contenedores, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: f9b54dbc9fc7c79bdb9b9aa106d5811401c365f3
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380469"
---
# <a name="linux-containers-on-windows-10"></a>Contenedores de Linux en Windows 10

> [!div class="op_single_selector"]
> - [Contenedores de Linux en Windows](quick-start-windows-10-linux.md)
> - [Contenedores de Windows en Windows](quick-start-windows-10.md)

Este ejercicio a través de la creación y ejecución de contenedores de Linux en Windows 10.

En este inicio rápido llevará a cabo:

1. Instalado Docker para Windows
2. Ejecutar un contenedor simple de Linux con contenedores de Linux en Windows (LCOW)

Este inicio rápido es específico de Windows10. En la tabla de contenido en el lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de que se cumplen los requisitos siguientes:
- Un sistema de equipo físico que ejecuta Windows 10 Professional o Enterprise con Fall Creators Update (versión 1709) o posterior
- Asegúrese de que [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) está habilitado.

***Aislamiento de Hyper-V:*** Los contenedores de Linux en Windows requieren aislamiento de Hyper-V en Windows 10 para ofrecer a los desarrolladores la el kernel de Linux adecuado para ejecutar el contenedor. Más acerca de Hyper-V aislamiento puede encontrarse en la página de [los contenedores de Windows sobre](../about/index.md) .

## <a name="install-docker-for-windows"></a>Instalar a Docker para Windows

Descarga [Docker para Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) y ejecuta el programa de instalación (se le pedirá que inicie sesión. Crear una cuenta si no tienes uno ya). Se encuentran disponibles [instrucciones de instalación detalladas](https://docs.docker.com/docker-for-windows/install) en la documentación de Docker (en inglés).

> Si ya tienes instalado de Docker, asegúrate de que tienes la versión 18.02 o posterior para admitir LCOW. Comprobar si ejecuta `docker -v` o comprobar *Acerca de Docker*.

> La opción 'características experimentales' en *> de configuración de Docker Daemon* debe estar activada para ejecutar contenedores LCOW.

## <a name="run-your-first-lcow-container"></a>Ejecutar el primer contenedor LCOW

Para este ejemplo, se implementará un contenedor BusyBox. En primer lugar, intenta ejecutar una imagen de "Hello World" BusyBox.

```console
docker run --rm busybox echo hello_world
```

Ten en cuenta que esto devuelve un error cuando se intenta extraer la imagen de Docker. Esto ocurre porque Dockers requiere una directiva a través de la `--platform` marca para confirmar que el sistema operativo host y la imagen se comparan correctamente. Dado que la plataforma de forma predeterminada en el modo de contenedor de Windows es Windows, agrega un `--platform linux` marca para extraer y ejecutar el contenedor.

```console
docker run --rm --platform linux busybox echo hello_world
```

Una vez que se ha extraído la imagen con la plataforma indica, el `--platform` marca ya no es necesario. Ejecuta el comando sin él para probarlo.

```console
docker run --rm busybox echo hello_world
```

Ejecutar `docker images` para devolver una lista de imágenes instaladas. En este caso, las imágenes de Windows y Linux.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Extra: Consulta correspondiente de Docker [blog publicar](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) acerca de cómo ejecutar LCOW.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtén información sobre cómo crear una aplicación de muestra](./building-sample-app.md)