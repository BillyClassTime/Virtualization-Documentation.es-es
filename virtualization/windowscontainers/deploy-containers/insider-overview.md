---
title: Usar contenedores con el programa Windows Insider
description: Obtenga información sobre cómo empezar a usar contenedores de Windows con el programa Windows Insider
keywords: Docker, contenedores, Insider, Windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909895"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Usar contenedores con el programa Windows Insider

Este ejercicio te llevará por la implementación y el uso de la función de contenedor de Windows en la última compilación para Insider de Windows Server desde el programa Windows Insider Preview. Durante este ejercicio, tendrás que instalar el rol de contenedor e implementar una edición de vista previa de las imágenes de sistema operativo base. Si necesitas familiarizarte con los contenedores, encontrarás esta información en [Acerca de los contenedores](../about/index.md).

## <a name="join-the-windows-insider-program"></a>Únete al Programa Windows Insider

Para ejecutar la versión de Insider de los contenedores de Windows, debe tener un host que ejecute la última compilación de Windows Server desde el programa Windows Insider o la última compilación de Windows 10 desde el programa Windows Insider. Únase al [programa Windows Insider](https://insider.windows.com/GettingStarted) y revise los términos de uso.

> [!IMPORTANT]
> Debe usar una compilación de Windows Server desde el programa de Windows Server Insider Preview o una compilación de Windows 10 desde el programa de Windows Insider Preview para usar la imagen base que se describe a continuación. Si no estás utilizando una de estas compilaciones, el uso de estas imágenes base dará como resultado errores al iniciar un contenedor.

## <a name="install-docker"></a>Instalar Docker

Si aún no tiene Docker instalado, siga la guía de [Introducción](../quick-start/set-up-environment.md) para instalar Docker.

## <a name="pull-an-insider-container-image"></a>Extracción de una imagen de contenedor de Insider

Al ser parte del programa Windows Insider, puede usar las últimas compilaciones para las imágenes base.

Para recuperar la imagen base de Insider de Nano Server, ejecuta lo siguiente:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Para recuperar la imagen base de Windows Server Core Insider, ejecuta lo siguiente:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

Las imágenes base "Windows" y "IoTCore" también tienen una versión de Insider que está disponible para la extracción. Puede obtener más información sobre las imágenes de la base de Insider disponibles en el documento de [imágenes base del contenedor](../manage-containers/container-base-images.md) .

> [!IMPORTANT]
> Lea el [CLUF](../images-eula.md ) de la imagen de sistema operativo de contenedores de Windows y los [términos de uso del](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)programa Windows Insider.
