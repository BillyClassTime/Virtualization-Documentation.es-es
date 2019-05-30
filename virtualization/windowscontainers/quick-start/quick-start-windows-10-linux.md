---
title: Contenedores de Windows y Linux en Windows 10
description: Inicio rápido de implementación de contenedores
keywords: acoplador, contenedores, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 91031f9394cb3fcb1af6c4813f8805ad6f79bf8c
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681105"
---
# <a name="linux-containers-on-windows-10"></a>Contenedores de Linux en Windows 10

> [!div class="op_single_selector"]
> - [Contenedores de Linux en Windows](quick-start-windows-10-linux.md)
> - [Contenedores de Windows en Windows](quick-start-windows-10.md)

El ejercicio recorrerá la creación y la ejecución de contenedores Linux en Windows 10.

En este inicio rápido, podrás realizar lo siguiente:

1. Instalar el escritorio del acoplador
2. Ejecución de un simple contenedor de Linux con contenedores Linux en Windows (LCOW)

Este inicio rápido es específico de Windows10. Puede encontrar documentación adicional de inicio rápido en la tabla de contenido, en la parte izquierda de esta página.

## <a name="prerequisites"></a>Requisitos previos

Asegúrate de cumplir los siguientes requisitos: <<<<<<< HEAD
- Un sistema informático físico con Windows 10 Professional o Enterprise con Windows otoño Update (versión 1709) o posterior
- Asegúrese [de que Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) está habilitado.
=======
- Un sistema informático físico con Windows 10 Professional, Windows 10 Enterprise o Windows Server 2019 versión 1809 o posterior
- Asegúrese [de que Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) está habilitado.
>>>>>>> origen o maestra

***Aislamiento de Hyper-V:*** Los contenedores de Linux en Windows requieren el aislamiento de Hyper-V en Windows 10 para proporcionar a los desarrolladores el kernel de Linux adecuado para ejecutar el contenedor. Encontrará más información sobre el aislamiento de Hyper-V en la página [acerca de los contenedores de Windows](../about/index.md) .

## <a name="install-docker-desktop"></a>Instalar el escritorio del acoplador

Descarga el [escritorio](https://store.docker.com/editions/community/docker-ce-desktop-windows) del acoplador y ejecuta el instalador (se te pedirá que inicies sesión. Crear una cuenta si aún no la tiene. Se encuentran disponibles [instrucciones de instalación detalladas](https://docs.docker.com/docker-for-windows/install) en la documentación de Docker (en inglés).

> Si ya tiene instalado un acoplador, asegúrese de que tiene la versión 18,02 o posterior para admitir LCOW. Comprobar ejecutando `docker -v` o comprobando *acerca*del acoplador.

> La opción ' características experimentales ' del *demonio de configuración del acoplador >* debe estar activada para poder ejecutar contenedores LCOW.

## <a name="run-your-first-lcow-container"></a>Ejecutar el primer contenedor LCOW

Para este ejemplo, se implementará un contenedor BusyBox. En primer lugar, intente ejecutar una imagen de BusyBox "Hola a todos".

```console
docker run --rm busybox echo hello_world
```

Tenga en cuenta que esto devuelve un error cuando el acoplador intenta extraer la imagen. Esto se debe a que los acopladores necesitan una `--platform` Directiva a través de la bandera para confirmar que el sistema operativo de la imagen y el host coinciden adecuadamente. Como la plataforma predeterminada en el modo contenedor de Windows es Windows, `--platform linux` agregue una marca para extraer y ejecutar el contenedor.

```console
docker run --rm --platform linux busybox echo hello_world
```

Una vez que se ha extraído la imagen con la plataforma `--platform` indicada, la bandera ya no es necesaria. Ejecute el comando sin él para probarlo.

```console
docker run --rm busybox echo hello_world
```

Ejecutar `docker images` para devolver una lista de las imágenes instaladas. En este caso, tanto la imagen de Windows como la de Linux.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonificación: vea la entrada de [blog](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) correspondiente al Docker en Running LCOW.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre cómo crear una aplicación de ejemplo](./building-sample-app.md)
